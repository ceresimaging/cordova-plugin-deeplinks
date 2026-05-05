# Change Log

All notable changes to this project will be documented in this file.
See [Conventional Commits](https://conventionalcommits.org) for commit guidelines

<a name="2.0.0"></a>
# 2.0.0 (2026-04-30)


### BREAKING CHANGES

* Plugin now requires Node >=20.17. Hooks rely on `fs.mkdirSync({recursive:true})` and `fs.rmSync({recursive:true,force:true})` directly instead of mkpath/rimraf/shelljs, and `plist` 4.0.0 is an ESM module loaded via Node's stable `require(esm)` interop (default-on in Node 20.17+).


### Features

* Drop `cordova-lib`, `mkpath`, `rimraf`, `shelljs` from dependencies; replace internal usage with Node 20 fs built-ins. Closes 8 transitive Dependabot alerts (cordova-common@<5, dep-graph, globby@<10, fast-glob, micromatch, braces, underscore, uuid@3, minimatch ReDoS).
* Bump `xcode` 2 → 3.0.1, `glob` 7 → 13.0.6, `plist` 1 → 4.0.0, `xml2js` 0.4 → 0.6.2 (fixes prototype-pollution CVE-2023-0842), `node-version-compare` → 1.0.3.
* Pin all dependency versions exactly (no `^`/`~`).


### Notes

* `cordova-lib` was unreferenced from the plugin's local `node_modules` — `context.requireCordovaModule` resolves from the host Cordova install. No runtime behavior change from the drop.
* Two moderate `uuid <14.0.0` alerts remain (GHSA-w5hq-g745-h8pq), inherited from `xcode@3.0.1`'s pinned `uuid@^7.0.3`. The advisory's vulnerable code path (`buf` parameter to v3/v5/v6 generators) is not reachable from xcode's v4-UUID-only usage, and uuid@14 is ESM-only so xcode (CommonJS) cannot consume it without an upstream rewrite. Re-evaluate when a maintained xcode release adopts uuid@14 or later.



<a name="1.2.0"></a>
# 1.2.0 (2026-01-30)

Ceres-internal release; never tagged on the public registry. Consumed by avian via `github:ceresimaging/cordova-plugin-deeplinks#release/1.2.0`.


### Bug Fixes

* **ios:** Skip entitlements rename for cordova-ios 8.x compatibility (54b24cd)
* **ios:** Detect project name from `.xcodeproj` folder for cordova-ios 8 (when platform folder is always "App") (da4aa93)
* **android:** Add `android:exported="true"` to deep-link intent filters for Android 12+ (133a0ec)
* Update repo links and add `.gitignore` (95ef7eb)



<a name="1.1.0"></a>
# [1.1.0](https://github.com/e-imaxina/cordova-plugin-deeplinks/compare/v1.0.1...v1.1.0) (2018-04-06)


### Bug Fixes

* Use ES5 syntax to support older android versions (5.1 and lower) ([3107b6e](https://github.com/e-imaxina/cordova-plugin-deeplinks/commit/3107b6e))


### Features

* Add support for cordova-android 7.0+ ([823e704](https://github.com/e-imaxina/cordova-plugin-deeplinks/commit/823e704))

<a name="1.0.1"></a>
## [1.0.1](https://github.com/e-imaxina/cordova-plugin-deeplinks/compare/v1.0.0...v1.0.1) (2018-03-07)


### Bug Fixes

* **hooks:** Invalid hook path (using old plugin name) ([1bb2aa4](https://github.com/e-imaxina/cordova-plugin-deeplinks/commit/1bb2aa4))



<a name="1.0.0"></a>
# 1.0.0 (2018-03-02)

**initial-release**: Initial release