# Using BrowserSync with WP Rig

WP Rig uses [BrowserSync](https://browsersync.io/) to enable synchronized browser testing. To take advantage of this feature, configure the `browserSync` wrapper settings in `./dev/config/themeConfig.js` to match your local development environment. The `proxyURL` value is the URL to the live version of your local site.

## Enabling HTTPS

In order to enable HTTPS with BrowserSync, you must supply a valid certificate and key with the Subject Alternative Name of `localhost`. Common Name has been deprecated since 2000 ([details](https://www.chromestatus.com/features/4981025180483584)).

WP Rig can generate a key and certificate valid for `localhost` for you with the command `npm run generateCert`. The key and certificates will be saved as `BrowserSync/wp-rig-browser-sync-key.key`, `BrowserSync/wp-rig-browser-sync-root-cert.crt` and `BrowserSync/wp-rig-browser-sync-cert.crt` in the WP Rig theme directory.

To use the generated key and certificate across multiple projects copy `BrowserSync/wp-rig-browser-sync-key.key`, `BrowserSync/wp-rig-browser-sync-root-cert.crt` and `BrowserSync/wp-rig-browser-sync-cert.crt` in the WP Rig theme directory to a global location, such as your home directory, uncomment and update the `keyPath` and `certPath` values in `dev/config/themeConfig.js` with the absolute paths for the key and certificate.

Then in future WP Rig projects you can simply set `https` to `true` and uncomment and update `keyPath` and `certPath` in `dev/config/themeConfig.js` with the paths of your existing key and certificate files rather than generating new ones for each project. Note that the certificate must be signed with an accompanying CA certificate and both certificates need to be trusted.

Unless `keyPath` and `certPath` are explicitly defined, the default paths of `./BrowserSync/wp-rig-browser-sync-key.key` and `./BrowserSync/wp-rig-browser-sync-cert.crt` in the WP Rig theme directory will be used. 

To use the default key and cert generated above, create new config settings in `./config/config.json` as follows:

```
{
  "dev": {
    "browserSync": {
      "live": true,
      "proxyURL": 'localwprig.com',
      "bypassPort": '8181',
      "https": true
  }
}
```

### Disable SSL certificate warnings

In addition, to disable SSL certificate warnings in the browser you will need to trust the certificate and CA certificate.

**For macOS:**
1. Open Keychain Access.
2. Drag the certificate file, which if using `npm run generateCert` to generate the certificates is saved to `BrowserSync/wp-rig-browser-sync-cert.crt`, into KeyChain access.
3. Double click on the new `WP Rig` certificate in the list, might also be labelled `localhost`.
4. Expand the `trust` menu.
5. Change the `When using this certificate:` setting to `Always Trust`.
6. Close the certificate dialogue.
    * You may be prompted for your computer password to save the setting.
    * If not, close the window and you will receive the prompt.
7. Repeat steps 3 through 6 for any other certificates.

Note that these steps only need to be done once for each key and certificate. The generated key and certificates expire after 5 years, at which point you will need to generate new ones and follow the steps above again.

**Screenshots of Keychain Access**

![Screenshots of the Keychain Access app](https://user-images.githubusercontent.com/1513046/46060089-cce54000-c12f-11e8-9f29-dedcfa3f9c52.png)

### Troubleshooting

**Enabling HTTPS with Firefox**
Navigate to `about:preferences` in Firefox, scroll down to the bottom of the page under Certificates -> View Certificates.

![firefox_add_cert_1](https://user-images.githubusercontent.com/819716/46054182-35262880-c114-11e8-9d0b-6b1157c6222d.png)

First, make sure the `Authorities` tab is active in the top nav bar.

![firefox_add_cert_2](https://user-images.githubusercontent.com/819716/46054183-37888280-c114-11e8-961d-36e295d3b392.png)

Second, click the middle bottom row Import -> navigate to directory of the of the certificates (wprig defaults to `/BrowserSync` select the root certificate `wp-rig-browser-sync-root-cert.crt` and import. If you are unable to select the certificate file, click the `options` tab to make sure `Certificate` file type is selected.

![firefox_add_cert_wprig_3](https://user-images.githubusercontent.com/819716/46054743-9d760980-c116-11e8-9317-1bb539bdc26a.png)
