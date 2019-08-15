# Updating to Gulp 4

Gulp 4 uses an updated CLI (Command Line Interface). If the computer you are using already has Gulp installed, there is a good chance you have an older version of the CLI and you will encounter errors when trying to run WP Rig.
## Updating the Gulp 4 CLI
To update the Gulp CLI to work with Gulp 4, run the following commands in the command line terminal:
```
# Uninstall Gulp globally:
npm uninstall gulp -g

# Install the latest version of the Gulp 4 CLI globally:
npm install gulpjs/gulp-cli -g
```
You may have to run `npm install` again from the WP Rig directory to ensure Gulp 4 is installed and ready to run.
