# CSS in WP Rig

## Table of contents:
- [WP Rig CSS philosophy](#wp-rig-css-philosophy)
  - [Layouts](#layouts)
- [File structure and CSS modules](#file-structure-and-css-modules)
- [Modern CSS via PostCSS](#modern-css-via-postcss)
  - [CSS Grid](#css-grid)
  - [Custom Properties (variables)](#custom-properties)
  - [Custom Media (queries)](#custom-media)
  - [Nesting](#nesting)
- [Why no Sass support?](#why-no-sass-support)
- [Progressive loading of CSS](#progressive-loading-of-css)
- [Code linting](#code-linting)
## WP Rig CSS philosophy
WP Rig promotes modern CSS best-practices by enabling developers to write and use modern CSS today and take advantage of upcoming CSS features even when these are not yet available in browsers. WP Rig uses pure CSS markup without pre-processors. The CSS provided by WP Rig follows these principles:
- Accessible 
- Mobile-first
- Modular
- Performance optimized
- Compliant with [WordPress Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/css/).
### Layouts
WP Rig uses the modern layout modules [CSS grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout) and [flex](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox) to support a minimalist HTML structure. The mobile-first layout serves as the baseline across all screen widths, and two-dimensional layouts are served as a progressive enhancement. As a result, browsers without grid/flex support are served with the mobile layout.

The layouts in WP Rig follow these design principles:
- Mobile layout for all screen sizes as baseline and fallback for older browsers.
- Two-dimensional layouts using CSS grid.
- One dimensional block/list-based displays using flex.
- In-content wrapping using float and clear.
## File structure and CSS modules
The file structure of CSS in WP Rig is modular with a strong focus on performance. This modular nature of the CSS files makes working with CSS easier, and improves performance thanks to progressive loading. Here’s how CSS works out of the box: 
- In WP Rig, `style.css` found in the root folder serves only as a container for the required theme configuration data. 
- All editable CSS is found under `assets/css/src/`. The contents of this folder are linted to [WordPress Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/css/) using [stylelint](https://stylelint.io/), minified, and output to `assets/css/` by the build process.
- `global.css` serves as the main stylesheet. This file is loaded in the `<head>` section of every template.
- All other files without an `_` prefix are loaded conditionally by their respective components (see below).
- Files prefixed by a `_` (eg `_blocks.css`) are _partials_, compiled into their parent files using `@import` rules. PostCSS imports the contents of these partials during processing to become part of the parent file.
- [Editor styles](https://wordpress.org/gutenberg/handbook/designers-developers/developers/themes/theme-support/#editor-styles) are found in `assets/css/src/editor/editor-styles.css`.

This modular file structure has several benefits including 
- discoverability (rules are grouped based on their function and/or module and easy to find)
- performance (files are only loaded when used)
- caching and bandwidth (individual files can be updated without the entire set of styles being updated)
CSS can be modified, deleted, and/or added to all existing files in `assets/css/src/`. When files are removed or new files are added to this folder, they need to be registered with the theme for use and called from the appropriate component. See [Progressive Loading of CSS](#progressive-loading-of-css) for further details.
## Modern CSS via PostCSS
All CSS is processed through [PostCSS](http://postcss.org/) and leveraging [postcss-preset-env](https://preset-env.cssdb.org/) to allow the use of modern and future CSS markup like [custom properties (variables)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables). postcss-preset-env support for modern features is configured by setting [stage](https://github.com/csstools/postcss-preset-env#options) and other configurations in `./config/config.default.json`:
```json
"styles": { 
  "stage": 3, 
  "importFrom": [ "_custom-media.css" ], 
  "features": { 
    "custom-media-queries": true, 
    "custom-properties": true, 
    "nesting-rules": true 
  } 
}
```
These settings can be modified in `./config/config.json`
PostCSS passes all CSS through [Autoprefixer](https://github.com/postcss/autoprefixer) to ensure backward compatibility based on [browserslist](https://github.com/browserslist/browserslist) settings found in `.browserslistrc`.
### CSS Grid
WP Rig uses CSS grid for two-dimensional layouts, in particular the overall layout of templates with sidebars and the front page template. PostCSS, postcss-preset-env, and Autoprefixer provide robust backwards compatibility for CSS grid, but there are some caveats. To learn more about how grid is handled by PostCSS, check out these resources:
- [CSS Grid in IE: CSS Grid and the New Autoprefixer](https://css-tricks.com/css-grid-in-ie-css-grid-and-the-new-autoprefixer/)
- [grid in postcss-preset-env documentation](https://github.com/csstools/postcss-preset-env#autoprefixer)
- [Grid autoplacement support in IE](https://github.com/postcss/autoprefixer#grid-autoplacement-support-in-ie)
### Custom Properties
Modern browsers support [Custom Properties, aka CSS variables](https://developer.mozilla.org/en-US/docs/Web/CSS/--*). Custom properties allow developers to declare a value once and reference this declaration throughout the stylesheet just like a regular variable in another coding language. Custom properties are scoped to the elements they are declared on, and follow the cascade, meaning a custom property can be declared, and then changed for a specific scope in a stylesheet.

A prototypical example of custom properties looks like this:
```css
:root { 
  --first-color: #488cff; 
  --second-color: #ffff8c; 
} 

#firstParagraph { 
  background-color: var(--first-color); 
  color: var(--second-color); 
}
```
WP Rig uses custom properties for a long list of values including font families and sizes, and colors. The custom properties are declarations in a CSS partial at  `./assets/css/src/_custom-properties.css`. This file is included at the top of `global.css`.

All custom properties in WP Rig are declared using the [`:root` selector](https://developer.mozilla.org/en-US/docs/Web/CSS/:root), a pseudo-class applying to “the root element of a tree representing the document”. This means the custom properties apply to the _entire_ document. The reason for this is backwards compatibility:
While modern browsers support custom properties, older browsers do not. PostCSS solves this problem by adding fallback property declarations any place a custom property is used, but this only works for custom properties declared at `:root`. The resulting CSS looks like this:

```css
body,
button, 
input, 
select, 
optgroup, 
textarea { 
  color: #333; 
  color: var(--global-font-color); 
  font-family: "Crimson Text", serif;; 
  font-family: var(--global-font-family); 
  font-size: calc(20 / 16 * 1rem); 
  font-size: var(--font-size-regular); 
  line-height: 1.4; 
  line-height: var(--global-font-line-height); 
}
```
This approach is deliberately conservative to provide a backwards-compatible starting point. Custom properties are quickly becoming a standard feature on the web, and developers are encouraged to use them with scoping when appropriate. 
## Custom Media
CSS custom properties can _not_ be used in media queries. For variables in media queries, the [Media Queries Level 5 specification introduces the `@custom-media` rule](https://drafts.csswg.org/mediaqueries-5/#custom-mq). This is bleeding-edge technology currently not supported by any browsers. 

[PostCSS Custom Media](https://github.com/postcss/postcss-custom-media) adds support for this upcoming feature, and WP Rig uses `@custom-media` with full backwards compatibility.

The custom media declarations are in a CSS partial at  `./assets/css/src/_custom-media.css`. Unlike custom properties, custom media references used in media queries in all CSS files are transformed by PostCSS into traditional media queries. This is done to prevent rule duplication. For developers this means you can safely use custom media in your themes and have WP Rig output functional CSS today as long as you declare the `@custom-media` rules in `./assets/css/src/_custom-media.css`.

When custom media gains proper support in modern browsers, WP Rig will be updated to incorporate this feature without transformations.
### Nesting
One popular feature of preprocessors like Sass and LESS is _nesting_, the ability to nest CSS rules inside one another for simpler cascade inheritance and less specificity. Thanks to the popularity of this feature, [CSS Nesting is now a level 1 module in the CSS spec](https://drafts.csswg.org/css-nesting-1/) meaning it will be implemented by modern browsers at some point in the not-too-distant future.

[PostCSS Nesting](https://github.com/jonathantneal/postcss-nesting) adds support for this upcoming feature following the [CSS nesting specification](https://drafts.csswg.org/css-nesting-1/). WP Rig has PostCSS Nesting built in meaning you can use CSS nesting in your stylesheets today.

The [CSS nesting specification](https://drafts.csswg.org/css-nesting-1/) has detailed information about how the feature works. The prototypical example is as follows:

```css
a, b { 
  color: red; 

  & c, & d { 
    color: white; 
  } 
} 

/* becomes */ 

a, b { 
  color: red; 
} 

a c, a d, b c, b d { 
  color: white; 
}
```
Note: CSS nesting is experimental, but thanks to PostCSS Nesting it is safe to use in WP Rig themes because the output is pure CSS (without nesting).
## Why no Sass support?
WP Rig 2.0 does not provide a build process for preprocessing SCSS ([Sass](https://sass-lang.com/)) files and partials. This is a deliberate decision in line with the overall philosophy of WP Rig: promote the latest best practices for progressive web content and optimization.

Sass is to CSS what jQuery is to JavaScript: A tool to prototype simpler/better/different development techniques for relatively rigid languages. And just like modern JavaScript and ESNext has adopted and built on many features from jQuery rendering jQuery less relevant, so has modern CSS done the same with Sass. 
The WP Rig team decided to drop support for Sass because all major features of Sass are now available in CSS, with fallbacks, via PostCSS:

Partials, imports, [custom properties (variables)](#custom-properties), [custom media](#custom-media), and [nesting](#nesting) are available to use in WP Rig via PostCSS. Advanced math is already part of the CSS spec thanks to `calc` and other operators. 

At present, the only major Sass features _not_ covered in WP Rig are _mixins_ and _extend/inherit_, this because they are presently not part of current or future CSS specifications. If you need these features, they can be added through PostCSS using [`postcss-mixins`](https://github.com/postcss/postcss-mixins) for mixins and either [`postcss-inherit`](https://github.com/GarthDB/postcss-inherit) or [`postcss-extend`](https://github.com/travco/postcss-extend) for extend/inherit. Including these features in WP Rig requires adding them as dev dependencies in `package.json` and altering the styles Gulp task in `./gulp/styles.js`.
## Progressive loading of CSS
WP Rig employs progressive loading of CSS through [in-body <link> tags](https://jakearchibald.com/2016/link-in-body/). Module-specific styles are held in module-specific stylesheets and loaded at module level. This is fundamentally different from how traditional WordPress themes handle CSS, and it has several advantages:
- The main stylesheet file size is reduced.
- Styles are only loaded if and when a module is present in the view.
- Stylesheets are associated with their modules making them easier to work with.
- Leverages HTTP/2 multiplexing and can be extended to include preloading, server push etc.
- Module-level stylesheets are cached and can be individually updated without forcing reload of all styles on the site.

As explained in [File structure and CSS modules](#file-structure-and-css-modules) above, only one stylesheet, `global.css` is always loaded by every template. All other stylesheets are loaded by their respective modules. This is done using the `wp_rig()->print_styles()` which takes a stylesheet handle as its parameter.
As an example, `content.css` is loaded by [`index.php` at line 19](https://github.com/wprig/wprig/blob/v2.0/index.php#L19):

```php
wp_rig()->print_styles( 'wp-rig-content' );
```
Each individual stylesheet is registered in the `get_css_files()` function found in `./inc/Styles/Component.php` under the `$css_files` array:

```php
$css_files = array(
  'wp-rig-global' => array( 
    'file' => 'global.min.css', 
    'global' => true, 
  ), 
  'wp-rig-comments' => array(
    'file' => 'comments.min.css', 
    'preload_callback' => function() { 
      return ! post_password_required() && is_singular() && ( comments_open() || get_comments_number() ); 
    }, 
  ), 
  'wp-rig-content' => array( 
    'file' => 'content.min.css', 
    'preload_callback' => '__return_true', 
  ), 
  'wp-rig-sidebar' => array( 
    'file' => 'sidebar.min.css', 
    'preload_callback' => function() { 
      return wp_rig()->is_primary_sidebar_active(); 
    }, 
  ), 
  'wp-rig-widgets' => array( 
    'file' => 'widgets.min.css', 
    'preload_callback' => function() { 
      return wp_rig()->is_primary_sidebar_active(); 
    }, 
  ),
  'wp-rig-front-page' => array( 
    'file' => 'front-page.min.css', 
    'preload_callback' => function() { 
      global $template; return 'front-page.php' === basename( $template ); 
    }, 
  ), 
);
```
Each entry in the `$css_files` array must follow the same convention:
```php
‘wp-rig-[label]’ => array(
  ‘file’ => ‘[filename].min.css’, // required
  ‘preload_callback’ => ‘[callback function or __return_true], // optional if file should be preloaded
  ‘global’ => true, // optional if file should be loaded in every template
)
```
The `preload_callback` value controls whether the stylesheet is preloaded in the `<head>` section of the document. This improves performance in some circumstances, but counteracts the performance benefits of using [in-body <link> tags](https://jakearchibald.com/2016/link-in-body/) because stylesheets are loaded _before_ they are used. For stylesheets that should _not_ be preloaded, use:
```php
`'preload_callback' => '__return_false'`
```
Tip: When preloading a stylesheet, use the console in Chrome developer tools to ensure no unnecessary stylesheets are loaded. A warning will appear letting you know if a stylesheet is preloaded but not used.
Note: This approach has raised some questions and concerns around the use of `wp_print_styles` and how to handle custom CSS from the Customizer. For those interested in further reading, these issues, along with some extended research, was addressed by @kevinhoffman in a comment to [issue #277](https://github.com/wprig/wprig/issues/277#issuecomment-459809352).
## Code linting
WP Rig provides CSS code linting to WordPress Coding Standards via [stylelint](https://stylelint.io/). Out of the box, this linting takes place during the build process and errors are referenced in the command line terminal. 
VS Code users can take full advantage of WP Rig’s stylelint configuration and get real-time coding assistance and format-on-save capability by enabling the [stylelint](https://marketplace.visualstudio.com/items?itemName=shinnn.stylelint) and [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) extensions and adding the following to Workspace Settings:
```json
{ 
  "prettier.stylelintIntegration": true, 
  "editor.formatOnSave": true 
}
```
This assumes VS Code was opened into the `wp-content/themes/wprig` folder as the workspace.
