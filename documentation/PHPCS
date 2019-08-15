# Enabling PHPCodeSniffer (PHPCS) in VS Code

WP Rig runs PHPCodeSniffer (PHPCS) as part of its build process to make sure all PHP complies with [WordPress Coding Standards](https://github.com/WordPress-Coding-Standards/WordPress-Coding-Standards). Any errors or warnings found by PHPCS are displayed in the command line terminal as the various WP Rig tasks run. 

To capture (and visualize) errors and warnings in VS Code as you are developing with WP Rig, you can activate the PHPCS extension and configure it to use the WP Rig settings:

In VS Code, select extensions and search for "phpcs" and install the free [PHPCS extension](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs) from the Visual Studio Marketplace.

In VS Code, open Workspace Settings (Ctrl+, on Windows, CMD+, on Mac) and add the following configuration settings:

```json
{
  "phpcs.enable": true,
  "phpcs.composerJsonPath": "[path]/[to]/composer.json",
  "phpcs.standard": "WordPress",
  "editor.renderWhitespace": "all"
}
```
Make sure the path in `phpcs.composerJsonPath` points at the `composer.php` file in your WP Rig folder _relative to the open project folder_ in VS Code. So if you opened `users/documents/mysite/` as your project folder and your WP Rig theme sits in `wp-content/themes/mytheme/`, the value should be `./wp-content/themes/mytheme/composer.json`.

With PHPCS configured in VS Code you will see green or red squiggly lines under any PHP which does not match the WordPress Coding Standards, and hovering your mouse over the highlighted code will open a tooltip telling you why PHPCS flagged this piece of code.
