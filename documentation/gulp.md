# Gulp in WP Rig

WP Rig uses a [gulp](https://gulpjs.com/) to assist in development and to generate and optimize code for production use. This article is an in-depth explanation of the gulp processes in WP Rig. If you want to know how to install WP Rig and get started see `README.md`.

## Background

As of version `2.0` WP Rig is meant to be edited as a development, or source, theme. Once development is complete WP Rig can generate a production version of the code, without the pieces needed for development. This is done to optimize both the development workflow as well as the production theme.

One of the slowest parts of the gulp process in version `1` of WP Rig was string replacement related tasks. With version `2` string replacement does not happen during development, only when the production theme is built.

Version `1` of WP Rig also used a `dev` subdirectory rather than a source and production structure. This approach is counter-intuitive to how WordPress themes are traditionally developed. Being able to edit any PHP file in WP Rig version `2`, without needing to know to only edit the ones in `dev`, will help lower the barrier to entry for developers new to WP Rig.

Version `2` of WP Rig also took the monolithic `gulpfile.babel.js` from version `1` and replaced it with more modular files, incorporating modern JavaScript best practices.

## Running `gulp` Tasks

The main gulp tasks, listed below, are run using [`npm scripts`](https://docs.npmjs.com/misc/scripts). This ensures that the gulp binary downloaded with npm is used.

- `npm run dev`
  - Processes all source files and watches files for subsequent changes.
- `npm run build`
  - Processes all source files once-time and does not watch for changes.
- `npm run bundle`
  - Creates a production version of the theme, saved to a separate directory in `wp-content/theme`, that does not include any unnecessary development files, such as asset source files.
- `npm run generateCert`
  - Generates `localhost` SSL certificates for use with the BrowerSync proxy server.
- `npm run translate`
  - Generates a `.pot` file based on the theme PHP files

gulp tasks not specifically mapped with npm script can still be run using the format `npm run dev -- <task_name>`, replacing `<task_name>` with the actual gulp task name. For example, `npm run dev -- images` will run the image task.

Available gulp tasks suitable to be run independently are:

- `images`
- `php`
- `scripts`
- `styles`
- `editorStyles`


Explanations of each gulp task are detailed in the next section of this article.

## gulp Tasks and Files

### gulpfile.babel.js

`gulpfile.babel.js` is the main gulp file. Using `gulpfile.babel.js` instead of `gulpfile.js` tells gulp to process the file with [Babel](https://babeljs.io/). This allows modern JavaScript to be used when writing the gulp functions themselves.

The purpose of `gulpfile.babel.js` is to define all available gulp tasks. This is done by exporting a named function for each task.

The main functions doing work are defined in separate files in the `gulp` directory and imported at the top of `gulpfile.babel.js`. Larger functions, using gulp `parallel` and `series` to define a specific order for the smaller functions, are then defined. Theme larger functions are then exported so that they become gulp tasks.

All of the smaller imported functions are exported as-is as well to allow them to be run as independent gulp tasks if desired.

## Other gulp files

### constants.js

`constants.js` is where constants for use in other gulp files are defined. Mainly this consists of file paths so that they do not need to be defined each time they are used and can easily be changed in one file.

### utils.js

`utils.js` is a collection of utility functions that other gulp functions make use of. If logic was repeated in multiple gulp functions it was moved here to avoid repetition.

### browserSync.js

`browserSync.js` defined [BrowserSync](https://www.browsersync.io/) related functions. Mainly, a BrowserSync proxy server. There is also a reload function which will refresh the BrowserSync server instance.

### generateCert.js

`generateCert.js` uses [`create-cert`](https://www.npmjs.com/package/create-cert) to programmatically generate SSL certificates for `localhost`. These certificates are required to use HTTPS with the BrowserSync server.

### images.js

`images.js` uses [`gulp-imagemin`](https://www.npmjs.com/package/gulp-imagemin) to optimized images.

### php.js

`php.js` reads PHP files and, optionally, uses [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer) to check them against [WordPress Coding Standards](https://github.com/WordPress-Coding-Standards).

If gulp is being run in production mode then the PHP files have the WP Rig name replaced and are saved to the production directory, in addition to the steps above.

### scripts.js

`scripts.js` reads JavaScript source files, uses [ESLint](https://eslint.org/) to check them against [WordPress Coding Standards](https://github.com/WordPress-Coding-Standards), transpiles the files with [Babel](https://babeljs.io/), and saves a minified version to the assets directory.

If gulp is being run in production mode then the optimized JavaScript files have the WP Rig name replaced and are saved to the production directory, in addition to the steps above.

### styles.js

`styles.js` reads CSS source files, uses [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer) to check them against [WordPress Coding Standards](https://github.com/WordPress-Coding-Standards), transpiles the files with [PostCSS](https://postcss.org/), and saves a minified version to the assets directory.

If gulp is being run in production mode then the optimized CSS files have the WP Rig name replaced and are saved to the production directory, in addition to the steps above.

### editorStyles.js

`editorStyles.js` runs the same process as `styles.js`, except it used `editor-styles.css`, which is enqueued for the block editor in the WordPress admin. Styles placed here are ones necessary for the block editor but not the front-end of the site.

### translate.js

`translate.js` reads the theme PHP files and uses `gulp-wp-pot` to generate a `.pot` file.

### watch.js

`watch.js` watches PHP, JS, CSS, image and configuration files. When they are changed the proper tasks are re-run and the BrowserSync server reloads.

### prodPrep.js

`prodPrep.js` is run if gulp is in production mode before other file processing tasks are run. The production directory is created and necessary files that are not otherwise processed with gulp, such as `readme.txt`, are copied to the production theme.

### prodFinish.js

`prodFinish.js` is run if gulp is in production mode after other file processing tasks are run and a `.zip` file of the production theme is created.
