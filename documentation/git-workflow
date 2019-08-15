# Recommended Git Workflow

WP Rig is constantly being improved to stay on top of the modern web and how it evolves. As such, it is crucial that you are able to pull in these latest improvements when building your own WP Rig-based theme. In order to accomplish that, WP Rig has to break conventions for how you develop your theme. In this article you will learn about what the recommended git workflow with WP Rig looks like.

## What is different?

When developing your theme based on WP Rig, you should continue to use the original "WP Rig" prefixes for all functions, classes, etc. Historically, when building a theme you would probably have used prefixes based on the name of your own theme. However, don't worry - with WP Rig you still get to brand your theme (including its code) with your own theme name. The build process includes an easy-to-use command that you can run to copy your development theme into a different folder in its production-ready version. And in that version all occurrences of "WP Rig" (or related prefixes) will have been replaced with your own ones.

## Why is that necessary?

When relying on other starter themes such as [Underscores (_s)](http://underscores.me), you will generate your base for developing a new theme once when you start. Underscores will take care of making all the replacements for you right away. This approach leads to one significant problem: You will never be able to update your theme with upstream changes that have been made to Underscores since you first generated your base, unless you're willing to go through a fully manual and time-consuming process.

WP Rig breaks with that paradigm: Prefixes remain "WP Rig", so you can easily pull in upstream changes that have been added to WP Rig after you started building your theme from it. This allows you to stay up to date also in the future. At the same time, your production-ready version of the theme will still use your own naming and prefixes as you envision it, so you don't actually lose anything.

## How does it work?

Here are some step-by-step guides on the recommended workflow with WP Rig. For the sake of this guide, let's assume your theme will be called "Morten", and your GitHub username is "wpdev".

### Setting up your own theme based on WP Rig

Here's what you need to do to set up your new theme based on WP Rig:

1. [Create a new GitHub repository](https://github.com/new) on your account called "morten".
2. Clone WP Rig into a local directory where you'll be developing in. Name that directory after your own theme followed by a suffix indicating it is the development version, e.g. "morten-dev" (`git clone git@github.com:wprig/wprig.git wp-content/themes/morten-dev`).
3. Go into that directory (`cd wp-content/themes/morten-dev`).
4. Delete the original WP Rig connection from your local repository (`git remote remove origin`).
5. Connect your local repository to your own GitHub project (`git remote add origin git@github.com:wpdev/morten.git`).
6. Re-add the connection to WP Rig as an upstream (`git remote add upstream git@github.com:wprig/wprig.git`).
7. Edit the configuration file `config/config.json` and add your own custom configuration based on `config/config.default.json` as necessary (most importantly make sure to update all values in the `theme` section). For an easy start, you could just copy the entire default configuration and then update it (`rm config/config.json && cp config/config.default.json config/config.json`).
8. Run `composer install` to install the necessary PHP dependencies.
9. Run `npm install` to install the necessary JavaScript dependencies.
10. Run `npm run build` to create a first build of your theme.
11. Create a first custom commit from your changes (`git add . && git commit -m "Customize configuration."`).
12. Push the changes to your own repository (`git push -u origin master`).
13. You're good to go now. Just make changes, commit them, and push them regularly.

### Creating a production-ready version of your theme

Remember: Don't bother manually replacing all the "WP Rig", "wp_rig_", etc. references in the codebase - for the development version of your theme, leave them as they are. Creating a production-ready version of your theme that you can then use on your website or release on wordpress.org) is fairly simple:

1. Run `npm run bundle` to create your production-ready version.
2. After the command has finished, a new directory with your theme slug (e.g. `morten`) will have been created at the same level as your development theme. That folder is ready as-is for you to deploy anywhere you like.

### Updating with upstream WP Rig changes

Every once in a while, you should update your development theme with the latest improvements that WP Rig has received in the meantime. WP Rig is generally developed in a backward-compatible way, with the exception of major versions (a.k.a. 2.0, 3.0, 4.0), which introduce breaking changes. So unless WP Rig has seen such a version bump, you can typically merge in upstream changes easily. Depending on how much you changed significantly in your theme, there is always a change for merge conflicts. Nonetheless, it's good to be aware of the possibility. As a general rule of thumb, pulling in changes regularly helps keeping the maintenance of updates low. If you only update once every 3 months, it's more likely you will run into a few merge conflicts, so you might need to dedicate one or two hours to it then. However, keep in mind that with the old paradigm of theme development you would never have updated at all, so your theme would have gotten out of date easily. Here is what you need to do to merge in the latest updates from WP Rig:

1. Update the latest upstream master branch (`git pull upstream master`).
2. Merge in the latest upstream changes to your own project's master branch (`git merge upstream/master`).
3. If any merge conflicts occur, you need to resolve them in order to complete the merge.
