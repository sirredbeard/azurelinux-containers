# syntax=docker/dockerfile:1.7
#
# Devolutions Terminal Linux build image.
# OCI-friendly: works with podman, buildah, and GitHub Actions buildx.
#
# Base tracks the floating .NET 11 preview SDK tag until 11 goes GA.
# Weekly rebuilds of this image pick up the newest 11.0-preview bits from MCR.
#
# Build (podman or docker):
#   podman build -t ghcr.io/sirredbeard/devolutions-terminal-build-container:local \
#     -f devolutions-terminal-build-container/Dockerfile \
#     devolutions-terminal-build-container
#
# Typical DT use:
#   podman run --rm -it -v "$PWD:/src:Z" -w /src \
#     ghcr.io/sirredbeard/devolutions-terminal-build-container:latest \
#     bash -lc 'dotnet test Devolutions.Terminal.slnx -c Release'

ARG DOTNET_SDK_IMAGE=mcr.microsoft.com/dotnet/sdk:11.0-preview
FROM ${DOTNET_SDK_IMAGE}

LABEL org.opencontainers.image.title="devolutions-terminal-build-container" \
      org.opencontainers.image.description="Build container for Devolutions Terminal on Linux (NativeAOT, Ghostty/Zig natives, tar/deb/rpm/AppImage)." \
      org.opencontainers.image.source="https://github.com/sirredbeard/azurelinux-containers" \
      org.opencontainers.image.url="https://github.com/sirredbeard/azurelinux-containers/tree/main/devolutions-terminal-build-container" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.vendor="Hayden Barnes" \
      org.opencontainers.image.base.name="mcr.microsoft.com/dotnet/sdk:11.0-preview"

ENV DEBIAN_FRONTEND=noninteractive \
    DOTNET_NOLOGO=1 \
    DOTNET_CLI_TELEMETRY_OPTOUT=1 \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1 \
    POWERSHELL_TELEMETRY_OPTOUT=1 \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# NativeAOT, packaging, Ghostty/Zig restore (pwsh), and ARM64 cross publish.
# Zig itself is fetched by DT's Restore-NativeLibraries.ps1 on first build.
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      wget \
      git \
      gnupg \
      apt-transport-https \
      software-properties-common \
      build-essential \
      clang \
      llvm \
      lld \
      lldb \
      make \
      cmake \
      pkg-config \
      python3 \
      python3-minimal \
      file \
      binutils \
      binutils-aarch64-linux-gnu \
      gcc-aarch64-linux-gnu \
      g++-aarch64-linux-gnu \
      libc6-dev-arm64-cross \
      zlib1g-dev \
      libicu-dev \
      libssl-dev \
      libkrb5-dev \
      liblttng-ust-dev \
      tar \
      gzip \
      xz-utils \
      unzip \
      zip \
      cpio \
      rpm \
      squashfs-tools \
      appstream \
      desktop-file-utils \
      jq \
      less \
      procps \
      locales \
    ; \
    sed -i 's/# *en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen; \
    locale-gen en_US.UTF-8; \
    rm -rf /var/lib/apt/lists/*

# PowerShell is required by native/Restore-NativeLibraries.ps1 and Ghostty build scripts.
RUN set -eux; \
    arch="$(dpkg --print-architecture)"; \
    case "$arch" in \
      amd64) pwsh_arch=x64 ;; \
      arm64) pwsh_arch=arm64 ;; \
      *) echo "unsupported arch: $arch" >&2; exit 1 ;; \
    esac; \
    PWSH_VERSION="$(curl -fsSL https://api.github.com/repos/PowerShell/PowerShell/releases/latest \
      | jq -r '.tag_name' | sed 's/^v//')"; \
    test -n "$PWSH_VERSION"; \
    curl -fsSL -o /tmp/powershell.tar.gz \
      "https://github.com/PowerShell/PowerShell/releases/download/v${PWSH_VERSION}/powershell-${PWSH_VERSION}-linux-${pwsh_arch}.tar.gz"; \
    mkdir -p /opt/microsoft/powershell/7; \
    tar -C /opt/microsoft/powershell/7 -xzf /tmp/powershell.tar.gz; \
    chmod +x /opt/microsoft/powershell/7/pwsh; \
    ln -sf /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh; \
    rm -f /tmp/powershell.tar.gz; \
    pwsh -NoLogo -Command '$PSVersionTable.PSVersion.ToString()'

# Confirm the floating SDK tag resolved to a usable toolchain.
RUN set -eux; \
    dotnet --info; \
    clang --version; \
    pwsh -NoLogo -Command 'Write-Host "pwsh ok"'; \
    python3 --version; \
    command -v rpmbuild; \
    command -v mksquashfs; \
    command -v aarch64-linux-gnu-objcopy; \
    command -v desktop-file-validate; \
    command -v appstreamcli

WORKDIR /src

# Default entry keeps the image useful as both CI job container and local shell.
CMD ["bash"]
