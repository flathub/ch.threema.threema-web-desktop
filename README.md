# Threema Desktop Electron App

This follows the upstream documentation in the README [1].
Note that this repository contains the actual web app as a submodule which we
need to declare as a separate dependency.

Further, we use `linux:deb` in various build scripts, but outside of
the actual `package` script, the `:deb` prefix doesn't do anything debian
specific.

## Packaging / Release Update Checklist

* [x] Make sure that `THREEMA_WEB_VERSION` environment variable matches the
  `version` field in the `package.json` of the `app/dependencies/threema-web`
  submodule (as described in the README).
* [x] Since the app uses Electron we use the official Flatpak Electron template.
  However, as described in detail here [2] we need to pre-fetch all dependencies,
  as node may not connect to the Internet within the build step.
  As the actual application and its `package-lock.json` is distributed over
  multiple files (and repos), we create the `generated-sources.json` file by
  cloning the repo with all submodules,
  and recursively (`-r`) collecting the info of all relevant dependency files
  using the flatpak-builder-tools [3]:

      git clone --recurse-submodules https://github.com/threema-ch/threema-web-electron.git
      flatpak-node-generator npm -r --electron-node-headers \
              package-lock.json

  We also need to add the electron headers, otherwise the postinstall of electron
  will try to fetch additional dependencies in the build step as well.

  *Note:* Make sure you run this on a clean source directory, otherwise the generator
  [may exclude some dependencies!](https://github.com/flatpak/flatpak-builder-tools/issues/354#issuecomment-1478518442)
* [x] Update the commit hash pointing to the new release
* [x] Make sure the metainfo.xml is up to date, patch it, if necessary.
* [x] Switch to a newer Freedesktop SDK if available.

[1]: https://github.com/threema-ch/threema-web-electron/
[2]: https://docs.flatpak.org/en/latest/electron.html
[3]: https://github.com/flatpak/flatpak-builder-tools/
