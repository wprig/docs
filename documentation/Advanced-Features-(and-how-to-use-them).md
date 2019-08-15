# Advanced Features (and how to use them)

WP Rig gives the developer an out of the box environment with support for modern technologies including ES2015, CSS grid, CSS custom properties (variables), and existing tools like Sass without making any configurations. Just write code and WP Rig handles the heavy lifting for you.

## WP Rig features
WP Rig takes a component-based approach to WordPress themes. Out of the box, the compiled theme uses `index.php` as the core template file for all views (index, archives, single posts, pages, etc).

The `/optional` folder holds optional template files that can be accessed via the [WordPress Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/). To activate these files, move or copy them into the root `wp-rig` theme folder. The `/optional` folder is ignored by the Gulp build process.

The separation of Pluggable and External features into their own folders allows the theme developer to swap out any feature for an external feature (AMP components) or non-php feature (JavaScript framework etc) without interfering with the core theme functions.

Pluggable functions and features (eg custom header, sliders, other interactive components) are separated into the `/pluggable` folder for easy access. When custom stylesheets and/or JavaScript files are needed, the pluggable component and its dependent files should be placed in a sub-folder to retain separation of concerns.

External features and add-ons are separated into the `/external` folder and are managed the same way as Pluggable functions.

Images and graphics are placed in the `/images` folder and are optimized automatically.

Global JavaScript files are placed in the `/js` folder and linted and optimized automatically. External JavaScript libraries are placed in the `/js/libs` folder. _These files are not linted or optimized by the Gulp process_.

Global stylesheets and stylesheets related to root-level php files are placed in the `/css` folder and are optimized automatically.

Content loop files are placed in the `/template-parts` folder.

`style.css` is required for WordPress to detect the theme but it is not used.

## Progressive Features

### Lazy-loading images
WP Rig [lazy loads](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video/) all images out of the box to improve performance. When lazy-loading images is enabled in the theme, the user will see a Theme Options feature in Customizer allowing them to toggle the feature off.
To disable this feature in the theme, comment out or remove the following line from `./functions.php`:

```php
require get_template_directory() . '/pluggable/lazyload/lazyload.php';
```

### AMP-ready
The theme generated by WP Rig is AMP-ready meaning it works hand-in-hand with the official AMP plugin and does not require further configuration to work with AMP features. The AMP plugin allows you to [opt-in to `amp` theme support via the plugin settings page](https://github.com/Automattic/amp-wp/wiki/Adding-Theme-Support) but you can also force enable AMP support in a theme by adding `add_theme_support( 'amp' );` in `./functions.php`.

#### AMP and custom JavaScript
When AMP support is enabled, JavaScript and other features are automatically disabled if the site admin has enabled the [official AMP plugin](https://en-ca.wordpress.org/plugins/amp/). Developers can selectively enable/disable features within the theme using the `wprig_is_amp()` conditional. For more see [Implementing Interactivity](https://github.com/Automattic/amp-wp/wiki/Implementing-Interactivity).

### Progressive loading of CSS
To further componentize the theme, WP Rig employs progressive loading of CSS through [in-body `<link>` tags](https://jakearchibald.com/2016/link-in-body/). Component-specific styles are held in component-specific stylesheets and loaded at component level. The `wprig_add_body_style()` in `./inc/template-functions.php` can be used to conditionally preload in-body stylesheets for improved performance.
This approach has several advantages:
- The main stylesheet file size is reduced
- Styles are only loaded if and when a component is present in the view.
- Stylesheets are associated with their components making them easier to work with.
- Leverages HTTP/2 multiplexing and can be extended to include server push etc.
- Component-level stylesheets are cached and can be individually updated without forcing reload of all styles on the site.

To improve the performance of the theme, progressively loaded stylesheets can be conditionally preloaded. This is done using the `wprig_add_body_style()` function in `./inc/template-functions.php`. When preloading a stylesheet, use the console in Chrome developer tools to ensure no unnecessary stylesheets are loaded. A warning will appear letting you know if a stylesheet is preloaded but not used.

### Modern CSS, custom properties (variables), autoprefixing, etc
All CSS is processed through [PostCSS](http://postcss.org/) and leveraging [postcss-preset-env](https://preset-env.cssdb.org/) to allow the use of modern and future CSS markup like [custom properties (variables)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables). Variables are defined in `./config/cssVariables` and applied to all CSS files as they are processed.
postcss-preset-env (previously cssnext) passes all CSS through Autoprefixer to ensure backward compatibility. [Target browsers](https://github.com/browserslist/browserslist) are configured under `browserlist` in `./config/cssVariables`.

### Modern layouts through CSS grid, flex, and float
The theme generated by WP Rig is mobile-first and accessible. It uses the modern layout modules CSS grid and flex to support a minimalist HTML structure.

For backward compatibility with browsers who do not support modern layout modules, WP Rig provides the mobile-first layout across all screen widths and serves two-dimensional layouts as a progressive enhancement.

The CSS philosophy of WP Rig breaks down as follows:
- Mobile layout for all screen sizes as fallback.
- Two-dimensional layouts using CSS grid.
- One dimensional block/list-based displays using flex.
- In-content wrapping using float and clear.