# azurelinux-containers

OCI container definitions I keep for builds and lab work. Same image should run under podman locally and in GitHub Actions.

## Containers

| Directory | Image | Purpose |
| --- | --- | --- |
| `devolutions-terminal-build-container` | `ghcr.io/sirredbeard/devolutions-terminal-build-container` | Linux build image for Devolutions Terminal (.NET 11 preview SDK, NativeAOT, packaging) |

## Layout

Each container lives in a top-level folder with a `Dockerfile` (optional `README.md`).

The weekly workflow scans the repo for every `Dockerfile` / `Containerfile`, builds it, pushes to GHCR, and keeps the last 3 versions of each package.

## Local build

```bash
podman build -t local/<folder>:dev -f <folder>/Dockerfile <folder>
```

## License

MIT. See [LICENSE](LICENSE).
