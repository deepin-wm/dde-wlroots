# dde-wlroots

A **statically linked** build of [wlroots](https://gitlab.freedesktop.org/wlroots/wlroots)
0.20.2, packaged for treeland and waylib.

The history is the real upstream wlroots history: the branch starts at the
pinned upstream release (see `UPSTREAM`), so `git blame` shows upstream authors,
and the treeland patches plus the CMake build and Debian packaging that turn it
into a build-time-only package sit on top as normal commits.

## Why static

treeland and waylib link the archive into their own shared objects, so
installing treeland does **not** pull in any library from this project. Only
`dde-wlroots-dev` is shipped, and only as a build dependency.

## Build

```sh
cmake -S . -B build -G Ninja -DCMAKE_INSTALL_PREFIX=/usr
cmake --build build
DESTDIR=$PWD/debian/tmp cmake --install build
```

Debian package: `dpkg-buildpackage -b` produces `dde-wlroots-dev`, which
installs:

- `/usr/lib/<multiarch>/libdde-wlroots.a`
- `/usr/include/wlr/` plus the generated `wlr/config.h`, `wlr/version.h` and the
  generated protocol headers (both under `/usr/include/wlr/` and flat in
  `/usr/include/`)
- `/usr/share/dde-wlroots/protocol/*.xml` — the wlroots-private protocol
  XMLs consumers generate code from
- `/usr/lib/<multiarch>/pkgconfig/dde-wlroots.pc`

## Consuming it

```cmake
pkg_check_modules(WLROOTS REQUIRED IMPORTED_TARGET dde-wlroots)
```

The `.pc` lists every dependency of the archive in `Requires`, so plain
`--cflags --libs` is already complete for static linking. It also exports the
metadata used by waylib to regenerate `wconfig.h`:

| variable | meaning |
| --- | --- |
| `wlroots_version`, `wlroots_version_{major,minor,patch}` | upstream version |
| `wlroots_features` | enabled `WLR_HAVE_*` features, space separated |
| `wlroots_protocoldir` | directory with the wlroots-private protocol XMLs |

## Updating from upstream

`UPSTREAM` records the pin. Because the upstream history is the history of this
repository, an update is a plain merge:

```sh
git fetch https://gitlab.freedesktop.org/wlroots/wlroots.git 0.21.0
git merge FETCH_HEAD
# then bump WLROOTS_VERSION* in CMakeLists.txt and UPSTREAM
```

Review `cmake/WlrootsSources.cmake`, `cmake/WlrootsProtocols.cmake` and
`cmake/WlrootsShaders.cmake` against the new upstream `meson.build` lists
afterwards.

## License

wlroots is MIT (see `LICENSE`). The CMake build and Debian packaging added
here are Apache-2.0 OR LGPL-3.0-only OR GPL-2.0-only OR GPL-3.0-only.
