# PHP Architecture in WP Rig

WP Rig changes the way WordPress themes are developed, not only on the client side, but also on the server side.

Historically, the common WordPress theme's PHP codebase has consisted of random PHP functions, scattered across different files, without too many architectural thoughts behind it. And while it is debatable whether themes should even have to contain as much server-side functionality as they do and not only focus on HTML, CSS and JavaScript (which the Gutenberg project might fix in the long run), the current state is that this is simply required.

WP Rig introduces several new patterns in how theme PHP code is structured and executed. These are not just for the sake of changing things, they solve actual problems. In other areas where a change wouldn't be more than of cosmetic nature, WP Rig stays as close as possible to the way themes have been developed in the past: As an example for that, WP Rig still uses a directory `inc` for its PHP functionality although nowadays it's more common to have this called `src` or similar - but changing that wouldn't solve a real problem, it would just be one more (in this case, admittedly simple) thing to get used to.

This article will first outline the goals of the WP Rig architecture and the problems it solves. It will then go into more detail what the architecture and API actually looks like and how to work with it. If you don't care about the background and just want to quickly start modifying and extending WP Rig's PHP code for your theme, you can skip the first section - although reading that may give you a better understanding and possibly provide you with ideas for other non-WP Rig WordPress projects as well.

## Goals of the WP Rig Architecture

### Clear Definition of Public API

WP Rig moves away from the function paradigm and organizes its code entirely in classes. Contrary to functions, which are always publicly available, class methods (i.e. "functions in classes") can have different visibility levels, for example to prevent calling a method from the outside of the class.

Most functions that themes include are internal to the theme, so they should not be exposed to the outside. Sure, if you have followed theme development for a while, you probably just know that you should not call a function like `mytheme_setup()` manually ever - as the theme already hooks it into `after_setup_theme`, thus has it be called automatically by WordPress. And its code is written in a way the function should definitely not be called again in the same request. However, this requires documentation and developers to know that. Nothing in the code restricts a developer from calling the function at any random point.

One might argue that calling this a problem is a bit overkill - but, to use another example, precisely the lack of proper definition of public vs private APIs causes WordPress core (which similar to themes mostly consists of public functions) to be so problematic with backward-compatibility. Almost every change that happens in WordPress core is in some way a breaking change. What is not considered a breaking change is probably so only because of the (in most cases true) assumption that nobody uses the respective function that's being changed in unexpected ways. In many other cases though, bug fixes get lost after endless discussions about things like "we probably should not fix this function, as someone might be using it in that way although it should not be used at all" - which brings us back here. One goal of WP Rig is have the code itself define that something is public or not public (of course still including actual documentation, like what you are reading now :) ).

### Separation of Concerns

WP Rig's classes are organized into so-called components. A component is responsible for a distinct feature of the theme. That can be an obvious feature literally like for example support for WordPress core's "Custom Header", but it can also be a feature in the more broad sense, such as the theme being fully compatible with "AMP" or the theme's focus on "accessibility" by providing a suite of generic accessibility fixes that WordPress itself does not include. Each of these components has a single main class, but can consist of as many classes as needed (although, at the point of writing, every component has in fact only the one main class). The components are then organized in different directories.

WordPress themes have historically done an okay-ish job of separating concerns. Aside from the terrible "you can dump this in your `functions.php`" approach, there are usually different files that contain the functions, often called for example `inc/template-tags.php` for functions that the template files need to use, or `inc/customize.php` for Customizer integration. However, as mentioned there is still use of the generic `functions.php` file, and even some functions themselves take care of multiple things that belong to different features. An example for that is the typical `mytheme_setup()` function that adds theme support for pretty much anything that theme support can be added for. That approach makes sense for one-off features, like `automatic-feed-links` or `title-tag`, things that every theme just needs to support and that are nothing more than a flag to enable. However, mixing in support for things like `custom-header`, `custom-background` or `editor-color-palette` is questionable, as each of these is a separate non-trivial feature with arguments (or at least is related to a separate feature like the editor). In the case of `custom-header`, adding theme support possibly even relies on other functions that need to be part of the theme.

Separating concerns helps keeping your codebase organized. This in turn helps navigating the codebase. Especially with WP Rig being a starter theme, it should be easy to find out which piece of code to navigate to when wanting to adjust a particular feature. For example, you might want to render the `custom-header` feature in a different way in your theme. Or you might not even want to support the `custom-header` feature at all. When looking at the historically used code organization of a WordPress theme, you'd never be able to find out where the code for that part was located. Of course, if you are a seasoned theme developer, you'd guess that it is in `functions.php`, but let's be honest, that's cheating because you already have made that experience often enough. In WP Rig, you instead have a "Custom Header" component, and thus also directory called `Custom_Header`, so it is obvious where to navigate. In fact, if you didn't want to support that feature, all you would have to do is delete the directory and remove [its instantiation in the `Theme` class](https://github.com/wprig/wprig/blob/v2.0/inc/Theme.php#L149) (more on the latter in the next section).

Another example that highlights the organization of the codebase is that WP Rig has a "Styles" component, but not a "Scripts" component. "Styles" is its own component since the theme's CSS files, while being granular, are not granular enough to apply per feature, as that would end up causing way too many CSS files to be present. Furthermore, CSS files in WP Rig are loaded in a special way to help performance, requiring extra PHP logic that makes the existence of a distinct "Styles" component more viable. JavaScript files however always exist to support one specific feature. For example there is the `navigation.js` script that makes the navigation menu more accessible - as such, it is part of the "Accessibility" component. Or there is the `lazyload.js` script taking care of lazy loading, making it part of the "Lazyload" component. Instead of dumping all `wp_enqueue_script()` calls of those scripts into one file, together with all the `wp_enqueue_style()` calls, JavaScript is loaded from the component that it is used for. This is precisely what the separation of concerns is about, organizing the codebase by its features, not by the WordPress functions it calls.

### Separating Invocation from Definition

Each of WP Rig's component main classes has an `initialize()` method that adds the necessary actions and filters. That means that the hooks are not automatically added just by including the file that the hook callback methods are part of - the theme explicitly needs to call the `initialize()` method. You might now ask "Why would I want to write extra code instead to call that method?" The reason is testability.

Preferably every piece of code in a project should be covered by tests that verify it works as expected. [WP Rig provides both unit and integration tests](https://github.com/wprig/wprig/tree/v2.0/tests/phpunit) to ensure that and to prevent unnoticed regressions. Particularly for unit tests, it is important that each unit (for example a PHP function or method) is tested in isolation without needing to consider side effects. By automatically having code executed on load of the same files that contain that code, this becomes impossible. In other words, invoking a function or method should not happen in the same location that also declares that function or method. In addition, the `initialize()` method of each component bundling all related `add_action()` and `add_filter()` calls ensures that there is a single entry point to hook in that component, i.e. integrate it with WordPress.

Don't worry, you still don't actually need to manually call the `initialize()` method of each component. Since each component must implement a specific interface (more on that later), the theme knows that each component has such an `initialize()` method and will invoke it automatically. As a matter of fact, [the `Theme` class has its own `initialize()` method](https://github.com/wprig/wprig/blob/v2.0/inc/Theme.php#L83) which calls the `initialize()` methods of all components. As such, this method is the single canonical entry point to the entire theme, integrating all theme components with WordPress.

## How to Work with the WP Rig Architecture

Before actually diving into the architecture, let's highlight that WP Rig uses [PHP namespaces](http://php.net/manual/en/language.namespaces.php) for its codebase. It also relies on [autoloading](http://php.net/manual/en/language.oop5.autoload.php), so every file containing a [class](http://php.net/manual/en/language.oop5.php), [interface](http://php.net/manual/en/language.oop5.interfaces.php) or [trait](http://php.net/manual/en/language.oop5.traits.php) should only contain that structure and nothing else. Such files should not be loaded anywhere manually, you can just rely on the autoloader. The root namespace for all code is `WP_Rig\WP_Rig` (remember that your own namespace will only be applied to your code when building the theme for production), and all code in that namespace (except tests) is located in the `inc` directory. From there on, sub-namespaces map to subdirectory names, and the class name itself must match the file name (followed by the `.php` extension of course), following the [PSR-4 standard](https://www.php-fig.org/psr/psr-4/). Make sure to follow these conventions for all files you might add to your theme based on WP Rig.

Something else to be aware of is that WP Rig uses function and method [parameter type hints](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) and [return type hints](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration) wherever possible. Since the theme requires PHP 7.0 as minimum version, it can use these features, and they are a simple means to make your code more error-proof.

### Working with Components

As mentioned earlier, the different PHP features of WP Rig are organized in so-called components. Whether you plan to adjust existing theme behavior or add an entirely new feature to the theme, you will work with components. For the sake of this guide, we will implement our own component while explaining how things work. Particularly, we will add a component that allows a user of the theme to toggle a Customizer setting to decide whether the theme's archive views should display excerpts (using `the_excerpt()`) or regular shortened content using the `<!-- more -->` tag (using `the_content()`). Maybe you even want to use this in your theme - the reason WP Rig does not include something like this is that it would still be too opinionated. WP Rig should be a starter theme and not get bloated by too many features people _might_ use (but likely more often would need to remove). But now, let's write our own component!

#### Scaffolding the Component Class

Every component has its own directory in `inc`, with the directory name matching the sub-namespace part, as explained in the previous paragraph. Let's call our component "Archive_Content". Based on that, the directory name must be `Archive_Content` and the namespace must be `WP_Rig\WP_Rig\Archive_Content`. Every component must have at least one class, the so-called component main class. That class should simply be called `Component` (note that the class name will still be unique because of the namespace, making the fully qualified class name to be something like `WP_Rig\WP_Rig\Archive_Content\Component`). The class must implement the [`WP_Rig\WP_Rig\Component_Interface` interface](https://github.com/wprig/wprig/blob/v2.0/inc/Component_Interface.php), thus you must at least implement the methods `get_slug()` and `initialize()`. Here is what this file could look like:

```php
<?php
/**
 * WP_Rig\WP_Rig\Archive_Content\Component class
 *
 * @package wp_rig
 */

namespace WP_Rig\WP_Rig\Archive_Content;

use WP_Rig\WP_Rig\Component_Interface;

/**
 * Class for allowing administrators to decide whether to display content or excerpt in archives.
 */
class Component implements Component_Interface {

	/**
	 * Gets the unique identifier for the theme component.
	 *
	 * @return string Component slug.
	 */
	public function get_slug() : string {
		return 'archive_content';
	}

	/**
	 * Adds the action and filter hooks to integrate with WordPress.
	 */
	public function initialize() {
		// TODO: Add actions and filters here.
	}
}

```

As you can see, `get_slug()` should simply return the slug of the component. This is used internally and should preferably match the component name in lowercase. In `initialize()` we will later add the actions and filters needed. Note the `use ...` statement below the namespace declaration: This line basically imports the `WP_Rig\WP_Rig\Component_Interface` so that we can use it in the namespace we are currently in. This is necessary because that interface's namespace is not the same as the namespace of our file. And apart from that, it also helps getting a quick overview of which language constructs from other locations are used in the file, i.e. which dependencies our file has. Also, keep in mind to thoroughly document your code at all times. This helps others in collaborating with you, and even if you're entirely on your own, it will help you remember your thoughts when you possibly come back to this code in two years. Especially if you're building a theme that should be public in any way, for example distributed on wordpress.org or sold through another platform, solid documentation is super important.

#### Adding Hooks

All hooks generally required to integrate a component with WordPress should be added in the `initialize()` method. The hook callbacks should be public methods in the same class, and as common pattern should be prefixed with either `action_` or `filter_`, depending on the respective hook type they are used for. The hook callbacks need to be public because otherwise WordPress will not be able to call them. If you read through the section about public APIs earlier in this article, you might now be asking "I thought interal hook callbacks should not be public!". Yes, it's true, and if the WordPress hook system supported calling protected or private methods, they indeed would not be. However, WP Rig's approach here still works because the component instances themselves are not available to the outside. As mentioned before, the `WP_Rig\WP_Rig\Theme` class holds all the component instances and calls their `initialize()` methods, but it does not expose these classes themselves to the outside. As such, essentially not even public methods in component classes are publicly usable, except for the hook that they are registered to directly. No worries, components can still contain methods that _are_ actually publicly usable and that you can use from outside locations such as PHP template parts. We will get to that in a bit. But first, let's see what the hook implementations for our component look like:

```php
<?php
/**
 * WP_Rig\WP_Rig\Archive_Content\Component class
 *
 * @package wp_rig
 */

namespace WP_Rig\WP_Rig\Archive_Content;

use WP_Rig\WP_Rig\Component_Interface;
use WP_Customize_Manager;
use function add_action;
use function __;

/**
 * Class for allowing administrators to decide whether to display content or excerpt in archives.
 */
class Component implements Component_Interface {

	/**
	 * Gets the unique identifier for the theme component.
	 *
	 * @return string Component slug.
	 */
	public function get_slug() : string {
		return 'archive_content';
	}

	/**
	 * Adds the action and filter hooks to integrate with WordPress.
	 */
	public function initialize() {
		add_action( 'customize_register', array( $this, 'action_register_customizer_control' ) );
	}

	/**
	 * Adds a Customizer setting and control for toggling usage of `the_content()` or `the_excerpt()` in archives.
	 *
	 * @param WP_Customize_Manager $wp_customize Customizer manager instance.
	 */
	public function action_register_customizer_control( WP_Customize_Manager $wp_customize ) {
		// Register the Customizer setting and ensure its value is a boolean.
		$wp_customize->add_setting(
			'archives_use_excerpt',
			array(
				'default'           => false,
				'sanitize_callback' => 'rest_sanitize_boolean',
			)
		);

		// Register the Customizer control for the setting, using a checkbox.
		$wp_customize->add_control(
			'archives_use_excerpt',
			array(
				'label'   => __( 'Use excerpts in archive views?', 'wp-rig' ),
				'section' => 'theme_options',
				'type'    => 'checkbox',
			)
		);
	}
}

```

Most of the newly added code should be familiar code if you have worked with the Customizer before. Similar as in the previous version of the code, note the new `use ...` statements. This time we import the `WP_Customize_Manager` class from WordPress core that is used here, plus several core functions. Contrary to classes, including statements for the functions is _not_ necessary here since namespaces will by default fall back to the global namespace (of which all WordPress core functions are a part of). However having these `use ...` statements in the file once again helps to quickly be able to tell which outside functions the file depends on.

With the above code, you already have the Customizer integration in place, and its value will be stored in a theme mod, so you can retrieve it with the WordPress function `get_theme_mod( 'archives_use_excerpt' )`. So now you can go into the [respective template file that displays post content](https://github.com/wprig/wprig/blob/v2.0/template-parts/content/entry_content.php) and adjust it as follows:

```php
<?php
/**
 * Template part for displaying a post's content
 *
 * @package wp_rig
 */
namespace WP_Rig\WP_Rig;

?>

<div class="entry-content">
	<?php
	if ( ! is_singular( get_post_type() ) && get_theme_mod( 'archives_use_excerpt' ) ) {
		the_excerpt();
	} else {
		the_content(
			// Same arguments as before.
		);
	}

	// Same code as before.
	?>
</div><!-- .entry-content -->

```

However, it would be much nicer to have a dedicated function to call for this, for example something like `using_archive_excerpts()`, that you can use from your template. We could of course just introduce a function for that and place it somewhere. However, that would go against our separation of concerns since it clearly belongs to our "Archive_Content" component. So let's look into how we can expose specific methods in our component to the outside.

#### Exposing Template Tags

WordPress typically calls functions that should be used from the template files of your theme "template tags". And there's no reason to change that naming convention. Historically, you would have put these as functions in a file called `template-functions.php` or `template-tags.php`. In WP Rig however, you can place them as methods alongside the rest of the code they actually integrate with, as part of the feature they are actually part of. While none of the component methods are exposed to the outside of the class by default, WP Rig contains a mechanism to work around that specifically for that purpose.

You might remember that each component has to implement the `WP_Rig\WP_Rig\Component_Interface` interface. A great thing about interfaces contrary is that it is possible for a class to implement multiple interfaces, contrary to abstract classes where a child can only extend a single parent class. WP Rig includes a `WP_Rig\WP_Rig\Templating_Component_Interface` interface, which is not mandatory to implement for each component, but only for those components that contain at least one method that should be exposed to the outside to be used in template files (again, these are called "template tags"). When a component main class implements this interface, it needs to include a method `template_tags()` which returns an associative array of `$template_tag_name => $template_tag_callback` pairs. The `$template_tag_callback` is the actual callback, which you should typically include as a public method on the class. The `$template_tag_name` is the name under which that template tag should globally be callable. This can in theory be different from the callback name, but it is recommended to keep the name consistent with the callback method, just for code readability. Any template tag made available like that can then be called via `wp_rig()->{$template_tag_name}()` - for example, for our component and the previously suggested name for the template tag we would use `wp_rig()->using_archive_excerpts()`. Internally, aggregating these template tags and exposing them via the `wp_rig()` function is handled by [the internal `WP_Rig\WP_Rig\Template_Tags` class](https://github.com/wprig/wprig/blob/v2.0/inc/Template_Tags.php), in combination with [`WP_Rig\WP_Rig\Theme`](https://github.com/wprig/wprig/blob/v2.0/inc/Theme.php). You don't necessarily need to worry about how this works - if you're interested, feel free to check out the code of the two classes and the `wp_rig()` function in more detail. Here is the full code for our component including exposure of the template tag:

```php
<?php
/**
 * WP_Rig\WP_Rig\Archive_Content\Component class
 *
 * @package wp_rig
 */

namespace WP_Rig\WP_Rig\Archive_Content;

use WP_Rig\WP_Rig\Component_Interface;
use WP_Rig\WP_Rig\Templating_Component_Interface;
use WP_Customize_Manager;
use function add_action;
use function __;
use function get_theme_mod;

/**
 * Class for allowing administrators to decide whether to display content or excerpt in archives.
 *
 * Exposes template tags:
 * * `wp_rig()->using_archive_excerpts()`
 */
class Component implements Component_Interface, Templating_Component_Interface {

	/**
	 * Gets the unique identifier for the theme component.
	 *
	 * @return string Component slug.
	 */
	public function get_slug() : string {
		return 'archive_content';
	}

	/**
	 * Adds the action and filter hooks to integrate with WordPress.
	 */
	public function initialize() {
		add_action( 'customize_register', array( $this, 'action_register_customizer_control' ) );
	}

	/**
	 * Gets template tags to expose as methods on the Template_Tags class instance, accessible through `wp_rig()`.
	 *
	 * @return array Associative array of $method_name => $callback_info pairs. Each $callback_info must either be
	 *               a callable or an array with key 'callable'. This approach is used to reserve the possibility of
	 *               adding support for further arguments in the future.
	 */
	public function template_tags() : array {
		return array(
			'using_archive_excerpts' => array( $this, 'using_archive_excerpts' ),
		);
	}

	/**
	 * Adds a Customizer setting and control for toggling usage of `the_content()` or `the_excerpt()` in archives.
	 *
	 * @param WP_Customize_Manager $wp_customize Customizer manager instance.
	 */
	public function action_register_customizer_control( WP_Customize_Manager $wp_customize ) {
		// Register the Customizer setting and ensure its value is a boolean.
		$wp_customize->add_setting(
			'archives_use_excerpt',
			array(
				'default'           => false,
				'sanitize_callback' => 'rest_sanitize_boolean',
			)
		);

		// Register the Customizer control for the setting, using a checkbox.
		$wp_customize->add_control(
			'archives_use_excerpt',
			array(
				'label'   => __( 'Use excerpts in archive views?', 'wp-rig' ),
				'section' => 'theme_options',
				'type'    => 'checkbox',
			)
		);
	}

	/**
	 * Determines whether archives should use excerpts instead of regular post content.
	 *
	 * @return bool Whether to use excerpts in archives.
	 */
	public function using_archive_excerpts() : bool {
		return (bool) get_theme_mod( 'archives_use_excerpt' );
	}
}

```

As you can see, a new `using_archive_excerpts()` method is added and then exported to the outside via the new `template_tags()` method that is added in combination with implementing `WP_Rig\WP_Rig\Templating_Component_Interface`. A good idea related to that is to document the available template tags in the source code, as seen in the doc block for the component class. Again, take note of how the additional dependencies (the interface and WordPress's `get_theme_mod()` function) have been added as `use ...` statements.

We now have the component class completed. Let's open [the post content template file](https://github.com/wprig/wprig/blob/v2.0/template-parts/content/entry_content.php) again and replace our previously used if condition with `! is_singular( get_post_type() ) && wp_rig()->using_archive_excerpts()`. What remains is that we need to make the theme aware of it.

#### Registering the Component Class

Making the theme use our new component class is very straightforward. Simply navigate to the [`WP_Rig\WP_Rig\Theme::get_default_components()` method](https://github.com/wprig/wprig/blob/v2.0/inc/Theme.php#L136) and add the new component in there. Note that the order of the components there matters in some cases, for example if similar hooks use a similar priority. Generally, a good approach to follow is to add more foundational components towards the top and more pluggable components or components that might even rely on other components towards the bottom of the list. For our case, let's append `WP_Rig\WP_Rig\Archive_Content\Component` to the end of the array. Note that since the class itself is in the `WP_Rig\WP_Rig` namespace, we can refer to our class relatively from that namespace, via `Archive_Content\Component`. Adding our component to the list is all we needed to do for the theme to handle it. It will call its `initialize()` method automatically now.

And that's it for our component! It might have been a bit of work getting this rather simple enhancement done and understanding how all the different parts connect. Once you become more familiar over time though, the architectural patterns used in WP Rig should boost your productivity significantly.
