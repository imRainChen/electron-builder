# electron-builder [![npm version](https://img.shields.io/npm/v/electron-builder.svg?label=latest)](https://yarn.pm/electron-builder) [![downloads per month](http://img.shields.io/npm/dm/electron-builder.svg)](https://yarn.pm/electron-builder) [![donate](https://img.shields.io/badge/Donate-Donorbox-green.svg)](https://www.electron.build/donate)
A complete solution to package and build a ready for distribution Electron app for macOS, Windows and Linux with “auto update” support out of the box.

* NPM packages management:
  * [Native application dependencies](http://electron.atom.io/docs/latest/tutorial/using-native-node-modules/) compilation (including [Yarn](http://yarnpkg.com/) support).
  * Development dependencies are never included. You don't need to ignore them explicitly.
* [Code Signing](https://electron.build/code-signing) on a CI server or development machine.
* [Auto Update](https://electron.build/auto-update) ready application packaging.
* Numerous target formats:
  * All platforms: `7z`, `zip`, `tar.xz`, `tar.lz`, `tar.gz`, `tar.bz2`, `dir` (unpacked directory).
  * [macOS](https://electron.build/configuration/configuration#MacOptions-target): `dmg`, `pkg`, `mas`.
  * [Linux](https://electron.build/configuration/configuration#LinuxBuildOptions-target): [AppImage](http://appimage.org), [snap](http://snapcraft.io), debian package (`deb`), `rpm`, `freebsd`, `pacman`, `p5p`, `apk`.
  * [Windows](https://electron.build/configuration/configuration#WinBuildOptions-target): `nsis` (Installer), `nsis-web` (Web installer), `portable` (portable app without installation), AppX (Windows Store), Squirrel.Windows.
* [Two package.json structure](https://electron.build/tutorials/two-package-structure) is supported, but you are not forced to use it even if you have native production dependencies.  
* [Build version management](https://electron.build/configuration/configuration#build-version-management).
* [Publishing artifacts](https://electron.build/publishing-artifacts) to GitHub Releases, Amazon S3 and Bintray.
* Pack in a distributable format [already packaged app](#pack-only-in-a-distributable-format).
* Separate [build steps](https://github.com/electron-userland/electron-builder/issues/1102#issuecomment-271845854).
* Build and publish in parallel, using hard links on CI server to reduce IO and disk space usage.
* [electron-compile](https://github.com/electron/electron-compile) support (compile for release-time on the fly on build).

| Question | Answer |
|--------|-------|
| “I want to configure electron-builder” | [See options](https://electron.build/configuration/configuration) |
| “I have a question” | [Open an issue](https://github.com/electron-userland/electron-builder/issues) or [join the chat](https://slackin.electron.build) |
| “I found a bug” | [Open an issue](https://github.com/electron-userland/electron-builder/issues/new) |
| “I want to donate” | [Donate](https://www.electron.build/donate) |

Real project example — [onshape-desktop-shell](https://github.com/develar/onshape-desktop-shell).

## Installation
[Yarn](http://yarnpkg.com/) is [strongly](https://github.com/electron-userland/electron-builder/issues/1147#issuecomment-276284477) recommended instead of npm.

`yarn add electron-builder --dev`

Platform specific `7zip-bin-*` packages are `optionalDependencies`, which may require manual install if you have npm configured to [not install optional deps by default](https://docs.npmjs.com/misc/config#optional).

## Boilerplates

* [electron-webpack-quick-start](https://github.com/electron-userland/electron-webpack-quick-start) — A bare minimum project structure to get started developing with [electron-webpack](https://github.com/electron-userland/electron-webpack). Recommended.
* [electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate) A boilerplate for scalable cross-platform desktop apps.
* [electron-react-redux-boilerplate](https://github.com/jschr/electron-react-redux-boilerplate) A minimal boilerplate to get started with Electron, React and Redux.
* [electron-boilerplate](https://github.com/szwacz/electron-boilerplate) A minimalistic yet comprehensive boilerplate application.

## Quick Setup Guide

1. Specify the standard fields in the application `package.json` — [name](https://electron.build/configuration/configuration#Metadata-name), `description`, `version` and [author](https://docs.npmjs.com/files/package.json#people-fields-author-contributors).

2. Specify the [build](https://electron.build/configuration/configuration#build) configuration in the `package.json` as follows:
    ```json
    "build": {
      "appId": "your.id",
      "mac": {
        "category": "your.app.category.type"
      }
    }
    ```
   See [all options](https://electron.build/configuration/configuration).

3. Add [icons](https://electron.build/icons).

4. Add the [scripts](https://docs.npmjs.com/cli/run-script) key to the development `package.json`:
    ```json
    "scripts": {
      "pack": "electron-builder --dir",
      "dist": "electron-builder"
    }
    ```
    Then you can run `yarn dist` (to package in a distributable format (e.g. dmg, windows installer, deb package)) or `yarn pack` (only generates the package directory without really packaging it. This is useful for testing purposes).

    To ensure your native dependencies are always matched electron version, simply add script `"postinstall": "electron-builder install-app-deps"` to your `package.json`.

5. If you have native addons of your own that are part of the application (not as a dependency), add `"nodeGypRebuild": true` to the `build` section of your development `package.json`.  
   :bulb: Don't [use](https://github.com/electron-userland/electron-builder/issues/683#issuecomment-241214075) [npm](http://electron.atom.io/docs/tutorial/using-native-node-modules/#using-npm) (neither `.npmrc`) for configuring electron headers. Use `electron-builder node-gyp-rebuild` instead.
   
6. Install the [required system packages](https://electron.build/multi-platform-build) if you are not on macOS 10.12+.

Please note that everything is packaged into an asar archive [by default](https://electron.build/configuration/configuration#Configuration-asar).

For an app that will be shipped to production, you should sign your application. See [Where to buy code signing certificates](https://electron.build/code-signing#where-to-buy-code-signing-certificate).

## CLI Usage

See [Command Line Interface](https://www.electron.build/cli).

## Programmatic Usage
See `node_modules/electron-builder/out/index.d.ts`. Typings for TypeScript is provided.

```js
"use strict"

const builder = require("electron-builder")
const Platform = builder.Platform

// Promise is returned
builder.build({
  targets: Platform.MAC.createTarget(),
  config: {
   "//": "build options, see https://goo.gl/ZhRfla"
  }
})
  .then(() => {
    // handle result
  })
  .catch((error) => {
    // handle error
  })
```

## Pack Only in a Distributable Format

You can use electron-builder only to pack your electron app in a AppImage, Snaps, Debian package, NSIS, macOS installer component package (`pkg`) 
and other distributable formats.

```
./node_modules/.bin/build --prepackaged <packed dir>
```

`--projectDir` (the path to project directory) option also can be useful.

## Community

[electron-builder](https://slackin.electron.build) on Slack (please use [threads](https://get.slack.help/hc/articles/115000769927-Message-threads)).
Public [archive](http://electron-builder.slackarchive.io) without registration.

## Further Reading
See [docs](https://electron.build).

## Debug

Set the [DEBUG](https://github.com/visionmedia/debug#windows-note) environment variable to debug what electron-builder is doing:
```bash
DEBUG=electron-builder,electron-builder:*
```
