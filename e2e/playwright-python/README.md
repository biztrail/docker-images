# Playwright e2e runner (Python)

**Image**: `ghcr.io/biztrail/library/e2e-runner-playwright-python:1.64.0`

**Base image**: `mcr.microsoft.com/playwright:v1.64.0-noble`

Job image for the Playwright e2e stack. The noble Playwright image already has Node, Yarn, and the 1.64.0 browsers under `/ms-playwright`. This layer adds the client the job otherwise installs on every run (Podman 4.9 + Compose), firebase-tools, and CPython 3.14.

Chromium is not reinstalled. `docker.io` is not installed: the job talks to the nested Podman socket and does not start a Docker daemon.

Compose still builds application images from their own Dockerfiles inside this client. Python here is the runner's interpreter, not the interpreter of those images.

Bump the image tag when the Dockerfile pins change. The publish workflow skips a platform that already has this tag.

## Pins

- Playwright `1.64.0` (browsers from the base image, `PLAYWRIGHT_BROWSERS_PATH=/ms-playwright`)
- firebase-tools `15.31.0`
- CPython `3.14` via `uv`, linked at `/usr/local/bin/python3.14`

## Installed packages

- podman
- docker-compose-v2 (`docker-compose` on `PATH`)
- crun, conmon, fuse-overlayfs, slirp4netns, uidmap, netavark, aardvark-dns, iptables, catatonit
- libpango, libharfbuzz, libcairo, libgdk-pixbuf, fontconfig, fonts-dejavu-core, fonts-liberation
- procps

`crun` is wrapped so it drops `process.oomScoreAdj` before create. Nested Podman cannot write `/proc/self/oom_score_adj`, and Docker Compose sends that field as `0`. The engine is set to `cgroupfs` with a file events logger.

## Usage

```bash
docker pull ghcr.io/biztrail/library/e2e-runner-playwright-python:1.64.0
```

```dockerfile
FROM ghcr.io/biztrail/library/e2e-runner-playwright-python:1.64.0
```

## Container Registry

View on [GitHub Packages](https://github.com/orgs/biztrail/packages/container/package/library%2Fe2e-runner-playwright-python)
