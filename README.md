# NCSA Mosaic

![NCSA Mosaic browser](http://github.com/downloads/alandipert/ncsa-mosaic/github.png "GitHub viewed with NCSA Mosaic")

NCSA Mosaic 2.7b9 — one of the first graphical web browsers, originally
developed at the [National Center for Supercomputing Applications (NCSA)](https://www.ncsa.illinois.edu/)
at the University of Illinois (1993–1997). This branch packages it for
modern Debian/Ubuntu systems.

Many thanks to [Sean MacLennan and Alan Wylie](https://web.archive.org/web/20120915154245/seanm.ca/mosaic/)
for the original porting work, and to Marc Andreessen, Eric Bina and the
rest of the NCSA team for kicking things off.

---

## Building from Source

### Platform

Debian 13 (trixie) or Ubuntu 22.04+. The build system requires GCC with
`-std=gnu89` support (all versions in Debian 10+ qualify).

### 1. Install build dependencies

```bash
sudo apt-get install \
  build-essential \
  libmotif-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libxmu-headers \
  libxpm-dev \
  libxmu-dev
```

> **Note for older docs / tutorials:** `libpng12-dev` and `x11proto-print-dev`
> are no longer available in Debian 13+. Use `libpng-dev` (libpng16) instead.
> No source changes are required — the code is compatible.

### 2. Build the binary

```bash
make linux
```

This compiles all internal libraries (`libwww2`, `libXmx`, `libhtmlw`,
`libnut`) and links the final binary at `src/Mosaic`.

### 3. Run

```bash
src/Mosaic
```

To open a specific URL on launch:

```bash
src/Mosaic https://example.com
```

---

## Building the Debian Package

### Quick build

```bash
dpkg-buildpackage -b -uc
```

The resulting `.deb` is placed one directory above the source tree:

```
../ncsa-mosaic_2.7b9-1_amd64.deb
```

### Install build dependencies via mk-build-deps (recommended)

```bash
sudo apt-get install devscripts equivs
sudo mk-build-deps --install --remove debian/control
```

### Install the package

```bash
sudo dpkg -i ../ncsa-mosaic_2.7b9-1_amd64.deb
sudo apt-get install -f   # resolve any missing runtime deps
ncsa-mosaic
```

The package installs:

| Path | Content |
|------|---------|
| `/usr/bin/ncsa-mosaic` | Main executable |
| `/etc/X11/app-defaults/Mosaic` | X application resources |
| `/etc/mosaic/mosaic-spoof-agents` | Browser spoof agent list |
| `/etc/mosaic/mosaic-user-defs` | User-facing defaults |
| `/usr/share/applications/edu.illinois.ncsa.mosaic.desktop` | Desktop entry |
| `/usr/share/metainfo/edu.illinois.ncsa.mosaic.metainfo.xml` | AppStream metadata |
| `/usr/share/icons/hicolor/<size>/apps/edu.illinois.ncsa.mosaic.png` | Icons (16–512 px) |

---

## What Was Changed for Debian 13 / GCC 14

The upstream source predates ANSI C. Two changes were made to
`makefiles/Makefile.linux` to compile cleanly on modern toolchains:

1. **`-std=gnu89` added to `CFLAGS`** — GCC 14 rejects implicit function
   declarations and implicit `int` return types by default; `gnu89` restores
   the permissive K&R behaviour without touching any source files.
2. **X11 include/library paths updated** — `/usr/X11R6/include` →
   `/usr/include`; `-L/usr/X11R6/lib` removed (standard paths suffice on
   modern Debian).

No `.c` or `.h` files were modified.

---

## CI / Jenkins

Automated builds run on
[jenkins.proxy.spojenet.cz](https://jenkins.proxy.spojenet.cz/job/Building/job/MXMosaic/)
for amd64, armhf and aarch64 across Debian buster → forky and Ubuntu
focal → resolute, driven by `debian/Jenkinsfile`.

---

## Licence

NCSA Mosaic is Copyright © 1993–1997 Board of Trustees of the University
of Illinois. Non-commercial use is free of charge. See `COPYRIGHT` for
the full terms.
