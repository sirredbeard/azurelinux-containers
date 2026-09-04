# devolutions-terminal-build-container

Linux build image for [Devolutions Terminal](https://github.com/Devolutions/devolutions-terminal).

## Base

`mcr.microsoft.com/dotnet/sdk:11.0-preview` until .NET 11 GA (expected November).

Weekly rebuilds pull the floating preview tag from MCR, so the image tracks the newest 11.0-preview SDK without editing the Dockerfile.

## What is inside

- .NET 11 preview SDK (from the base tag)
- clang / llvm / lld and the usual NativeAOT libs
- PowerShell 7 (`pwsh`) for `native/Restore-NativeLibraries.ps1` and the Ghostty build path
- git, python3, file, binutils, tar
- packaging: `rpm` / `rpmbuild`, `squashfs-tools` / `mksquashfs`, `ar`
- desktop metadata: `appstreamcli`, `desktop-file-validate`
- ARM64 cross: `gcc-aarch64-linux-gnu`, `binutils-aarch64-linux-gnu`, `libc6-dev-arm64-cross`
- GTK4 + libadwaita headers, runtime libs, and GObject introspection (Gir.Core shell)
- Qt6 base / widgets / wayland / declarative dev packages (Plasma shell spike later)

`pkg-config` must see `gtk4`, `libadwaita-1`, and `Qt6Core` / `Qt6Gui` / `Qt6Widgets` after the image builds. Image build fails closed if those modules are missing.

Zig is not preinstalled. DT downloads the pinned Zig version on first native restore.

AppImage still needs a pinned type-2 runtime on disk via `APPIMAGE_RUNTIME_FILE`. This image does not vendor that binary.

## Local build (podman)

```bash
cd ~/GitHub/azurelinux-containers
podman build \
  -t ghcr.io/sirredbeard/devolutions-terminal-build-container:local \
  -f devolutions-terminal-build-container/Dockerfile \
  devolutions-terminal-build-container
```

Docker works the same way. Swap `podman` for `docker`.

## Use with a DT checkout

```bash
cd ~/GitHub/devolutions-terminal
podman run --rm -it \
  -v "$PWD:/src:Z" \
  -w /src \
  ghcr.io/sirredbeard/devolutions-terminal-build-container:latest \
  bash -lc 'dotnet test Devolutions.Terminal.slnx -c Release'
```

NativeAOT + packages:

```bash
podman run --rm -it \
  -v "$PWD:/src:Z" \
  -w /src \
  -e APPIMAGE_RUNTIME_FILE \
  ghcr.io/sirredbeard/devolutions-terminal-build-container:latest \
  bash -lc 'scripts/Build-LinuxPackage.sh linux-x64 0.1.0 artifacts/packages all'
```

## GHCR

Published by the repo workflow as:

`ghcr.io/sirredbeard/devolutions-terminal-build-container`

Tags: `latest`, `sha-<git sha>`, `utc-YYYYMMDD-HHMMSS`.

Older tagged builds are pruned to the last three after each push. Untagged attestation rows older than that window are removed too.
