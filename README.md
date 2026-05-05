# cordova-plugin-deeplinks (Ceres fork)

Cordova plugin that opens your app from links the user clicks in the browser, mail, or other apps:

- [Universal Links on iOS](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content)
- [App Links / Deep Linking on Android](https://developer.android.com/training/app-links)

This is the **Ceres fork** of [`cj4luv/cordova-plugin-deeplinks`](https://github.com/cj4luv/cordova-plugin-deeplinks) (itself a fork of [`nordnet/cordova-universal-links-plugin`](https://github.com/nordnet/cordova-universal-links-plugin)). It carries Ceres-specific compatibility patches for modern Cordova toolchains and is maintained at [github.com/ceresimaging/cordova-plugin-deeplinks](https://github.com/ceresimaging/cordova-plugin-deeplinks).

## What this fork adds

- **cordova-ios 8.x compatibility** — skip the legacy entitlements rename when the iOS platform folder is the cordova-ios 8 default `App/`, and detect the project name from the `*.xcodeproj` folder rather than `config.xml`.
- **Android 12+ deep-link intent filters** — `android:exported="true"` is written on the generated intent filter, required since Android S.
- **Modernized dependency tree** — drops `cordova-lib`, `mkpath`, `rimraf`, and `shelljs` in favor of Node 20 built-ins; bumps `xcode`, `glob`, `plist`, `xml2js` to current majors. Closes the transitive Dependabot alerts that the upstream plugin's 2018-era deps were dragging into consuming projects.

See [CHANGELOG.md](CHANGELOG.md) for full per-release detail.

## Requirements

- **Node**: `>=20.17` (the plugin's hooks load `plist@4` via Node's stable `require(esm)` interop, which is default-on starting in 20.17)
- **Cordova**: `>=13`
- **cordova-ios**: `>=3.8` (tested on 8.0.x; the `App/` folder convention used by 8.x is required for the bundled patches)
- **cordova-android**: `>=4` (tested on 14.x and 15.x)
- **Xcode**: 16+ when targeting current iOS releases

## Installation

This fork is consumed via a GitHub ref, not npm. Add to your project's `package.json`:

```json
{
  "devDependencies": {
    "cordova-plugin-deeplinks": "github:ceresimaging/cordova-plugin-deeplinks#release/2.0.0"
  }
}
```

Or directly via the Cordova CLI:

```sh
cordova plugin add github:ceresimaging/cordova-plugin-deeplinks#release/2.0.0
```

Pin to a tag (`#release/2.0.0`) rather than a branch so the install is reproducible.

## Configuration

Configure supported hosts and paths in your Cordova `config.xml` inside a `<universal-links>` block. The plugin reads this at `cordova prepare` time to generate the iOS entitlements, the Android intent filters, and the `apple-app-site-association` / `assetlinks.json` web hook files.

```xml
<universal-links>
  <ios-team-id value="ABCDE12345" />
  <host name="app.example.com" scheme="https">
    <path url="/news" />
    <path url="/account/*" />
  </host>
</universal-links>
```

### `<host>`

Declares a domain the app should open from.

- `name` (required) — hostname. Wildcards like `*.example.com` are supported.
- `scheme` — `http` or `https`. Defaults to `http`. Use `https` for iOS Universal Links.
- `event` — JS event name to dispatch when the app launches from this host. If omitted, the JS callback is registered with `null`.

### `<path>`

Restricts which paths under a host trigger an app launch. Omit `<path>` to handle every URL on the host.

- `url` (required) — path relative to the host. `*` is allowed as a suffix or substring (e.g. `/news/*`, `*search*`). Query strings and fragments are not used for matching.
- `event` — JS event name for this specific path. Falls back to the host-level event, then to `null`.

### `<ios-team-id>`

Apple Developer Team ID, written into generated `apple-app-site-association` files. Without this, the generated files contain a placeholder you'd have to substitute manually before serving them.

## JavaScript API

The plugin exposes a global `universalLinks` object.

```js
universalLinks.subscribe('myEvent', (eventData) => {
  // eventData = { url, scheme, host, path, params, hash }
  console.log('Opened from', eventData.url);
});

universalLinks.unsubscribe('myEvent');
```

If you didn't set an `event` attribute in `config.xml`, subscribe with `null`:

```js
universalLinks.subscribe(null, (eventData) => { /* ... */ });
```

`eventData` shape:

| field | description |
|---|---|
| `url` | original launch URL |
| `scheme` | `http` or `https` |
| `host` | hostname |
| `path` | path component |
| `params` | query string parsed into `{ key: value }` |
| `hash` | fragment (after `#`) |

## Android web integration (assetlinks.json)

When `cordova prepare` runs, the plugin writes `ul_web_hooks/android/android_web_hook.html` with the `<link rel="alternate">` tags Google App Indexing expects. For Android 6.0+ App Links, you also need an `assetlinks.json` at `https://<your-host>/.well-known/assetlinks.json` containing your package name and the SHA-256 fingerprint of your signing cert. Generate the fingerprint with:

```sh
keytool -list -v -keystore <your.keystore> -alias <your-alias>
```

The plugin does not generate `assetlinks.json` — host it yourself. The `android_web_hook.html` it does generate is meant to be embedded in your site's `<head>` to support App Indexing.

## iOS web integration (apple-app-site-association)

After `cordova prepare`, look in `ul_web_hooks/ios/`. You'll find one file per host:

```
app.example.com#apple-app-site-association
```

Contents (with `<ios-team-id>` substituted):

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "ABCDE12345.com.example.app",
        "paths": ["/news", "/account/*"]
      }
    ]
  }
}
```

Upload the file to `https://<host>/.well-known/apple-app-site-association` (or the legacy `https://<host>/apple-app-site-association`). It must be served over HTTPS, with `Content-Type: application/json`, and **no redirects** — iOS won't follow them.

You also need to enable **Associated Domains** on the App ID in the Apple Developer console and add `applinks:<host>` entries to the iOS entitlements (the plugin writes these into `App.entitlements` automatically based on `config.xml`).

## Testing

### Android

From a connected device or emulator with the app installed:

```sh
adb shell am start -W -a android.intent.action.VIEW \
  -d "https://app.example.com/news" \
  <your.package.name>
```

If the link is configured in `config.xml` and `assetlinks.json` is verified, the app launches directly. Otherwise Android may show a chooser dialog.

### iOS (simulator)

```sh
xcrun simctl openurl booted https://app.example.com/news
```

For real-device testing, the `apple-app-site-association` file must be live on your domain — Apple's `swcd` daemon fetches it on app install. If links open Safari instead of your app, check Console.app on your Mac for `swcd` errors.

## Build and consume

This fork is internal to Ceres and is consumed by `avian/src-cordova` directly from GitHub. Releases are tagged on this repo as `release/X.Y.Z`. The full build chain inside Avian is:

1. `avian/src-cordova/package.json` references `github:ceresimaging/cordova-plugin-deeplinks#release/X.Y.Z`.
2. `npm install` in `src-cordova/` fetches this repo at the pinned tag.
3. `cordova plugin add` runs `hooks/beforePluginInstallHook.js`, which `npm install --production`s this plugin's own dependencies into `plugins/cordova-plugin-deeplinks/node_modules/`.
4. `cordova prepare` runs `hooks/iosBeforePrepareHook.js` (iOS entitlements rename + cordova-ios 8 detection) and `hooks/afterPrepareHook.js` (entitlements file generation, AndroidManifest intent filters, web hook files).

To cut a new release of this fork:

1. Branch from the previous release: `git checkout -b release/X.Y.Z` from `release/(X-1).Y.Z`.
2. Edit code, bump `version` in `package.json`, add a CHANGELOG entry.
3. Run `npm install && npm audit` and resolve fixable issues.
4. Smoke-test inside Avian via a `file:` reference before tagging:

   ```sh
   # in avian/src-cordova/package.json
   "cordova-plugin-deeplinks": "file:../../cordova-plugin-deeplinks"
   ```

   Then `cordova platform rm android ios && cordova platform add android ios && cordova prepare` and verify the generated `AndroidManifest.xml` and `App.entitlements`.
5. Commit, tag `release/X.Y.Z`, push branch and tag.
6. Update `avian/src-cordova/package.json` to point at the new tag and open a PR.

## Upstream attribution

The original plugin was written by Nikolay Demyankov for Nordnet Bank AB. The cordova 7+ adaptation was carried by `cj4luv`. This fork keeps the MIT license; see [LICENSE](LICENSE) for terms.

For issues with config syntax, JS API behavior, or web-integration steps that aren't Ceres-specific, the upstream documentation at [`nordnet/cordova-universal-links-plugin`](https://github.com/nordnet/cordova-universal-links-plugin) remains the most thorough reference.
