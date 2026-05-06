# AGENTS.md - Working AI Reference for MXMosaic

## Project Overview
**Type**: Debian Package (legacy C application)
**Purpose**: NCSA Mosaic 2.7b9 — one of the first graphical web browsers (1993–1997)
**Status**: Active packaging, upstream archived
**Repository**: git@github.com:Vitexus/MXMosaic.git
**AppStream ID**: `edu.illinois.ncsa.mosaic`
**Debian package**: `ncsa-mosaic` version `2.7b9-1`
**Jenkins pipeline**: https://jenkins.proxy.spojenet.cz/job/Building/job/MXMosaic/

## Key Technologies
- C (K&R / gnu89 standard) — old-style NCSA source code
- Motif (libXm) GUI toolkit
- Debian packaging (debhelper compat 13)
- AppStream metainfo

## Architecture & Structure
```
MXMosaic/
├── src/              # Main Mosaic binary source
├── libXmx/           # Motif helper library
├── libhtmlw/         # HTML widget library
├── libwww2/          # HTTP/FTP/Gopher network library
├── libnut/           # Utility library
├── libdtm/           # DTM (unused in standard build)
├── libnet/           # DTM network layer (unused)
├── makefiles/
│   └── Makefile.linux  # Linux build config (patched for Debian 13)
├── debian/
│   ├── control         # Build-Depends updated for trixie
│   ├── rules           # Custom build/install overrides
│   ├── changelog       # Version 2.7b9-1
│   ├── edu.illinois.ncsa.mosaic.desktop
│   ├── edu.illinois.ncsa.mosaic.metainfo.xml
│   ├── icons/          # PNG icons 16–512 px (from alrra/browser-logos)
│   └── Jenkinsfile     # CI pipeline (Building/MXMosaic)
└── AGENTS.md
```

## Build Dependencies (Debian 13 / trixie)
```
build-essential, libmotif-dev, libjpeg62-turbo-dev, libpng-dev,
libxmu-headers, libxpm-dev, libxmu-dev
```
Note: `libpng12-dev` and `x11proto-print-dev` are obsolete — use `libpng-dev`.

## Development Workflow

### Build locally
```bash
dpkg-buildpackage -b -uc
```

### Install build deps
```bash
sudo apt-get install build-essential libmotif-dev libjpeg62-turbo-dev \
  libpng-dev libxmu-headers libxpm-dev libxmu-dev
```

### Run the browser
```bash
src/Mosaic        # after make linux
ncsa-mosaic       # after dpkg install
```

## Key Patches Applied for Debian 13
- `makefiles/Makefile.linux`: `-std=gnu89` added to CFLAGS (GCC 14 rejects
  implicit function declarations by default); X include path changed from
  `/usr/X11R6/include` → `/usr/include`; removed stale `-L/usr/X11R6/lib`.
- No source code changes were required.

## AppStream / Desktop Integration
- Metainfo: `debian/edu.illinois.ncsa.mosaic.metainfo.xml`
  - Validates clean with `appstreamcli validate --pedantic --no-net`
- Desktop entry: `debian/edu.illinois.ncsa.mosaic.desktop`
- Icons: hicolor PNG at 16, 24, 32, 48, 64, 128, 256, 512 px
  - Source: https://github.com/alrra/browser-logos/tree/a94987f/src/archive/mosaic
  - Installed as `edu.illinois.ncsa.mosaic.png` in each hicolor size dir

## CI / Jenkins
- Folder: `Building`, job: `MXMosaic`
- URL: https://jenkins.proxy.spojenet.cz/job/Building/job/MXMosaic/
- Branch: `debianized`
- Jenkinsfile: `debian/Jenkinsfile`
- Builds for architectures detected from `debian/control` (Architecture: any
  → amd64, armhf, aarch64) across multiple Debian/Ubuntu distributions.

## Troubleshooting
- **Compile errors about implicit declarations**: ensure `-std=gnu89` is in
  `makefiles/Makefile.linux` CFLAGS — this code predates ANSI C.
- **Missing X11R6 paths**: update `xinc` and `xlibs` in `Makefile.linux`;
  modern Debian puts X headers/libs in standard system paths.
- **libpng API errors**: `png_set_message_fn` / `png_get_msg_ptr` were removed
  in libpng 1.4; the call is already inside `#ifdef SAM_NO` so it won't
  compile — do not remove that guard.
