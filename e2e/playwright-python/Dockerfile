# GitLab job image for the Playwright stack: biztrail/e2e-runner-playwright-python
#
#   ghcr.io/biztrail/library/e2e-runner-playwright-python:1.64.0
#
# The noble Playwright image already has Node, Yarn, and the 1.64.0 browsers
# under /ms-playwright. This layer adds the client the job otherwise apt-gets
# on every run (Podman 4.9 + Compose), firebase-tools, and CPython 3.14.
# Chromium is not reinstalled. docker.io is not installed: the job talks to
# the nested Podman socket, it does not start a Docker daemon.
#
# Compose still builds intranet-api from apps/api/Dockerfile inside this
# client. Python here is the runner's interpreter, not that image.

FROM mcr.microsoft.com/playwright:v1.64.0-noble

ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright \
    UV_PYTHON_INSTALL_DIR=/opt/python \
    UV_LINK_MODE=copy

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        podman \
        docker-compose-v2 \
        crun \
        conmon \
        fuse-overlayfs \
        slirp4netns \
        uidmap \
        netavark \
        aardvark-dns \
        iptables \
        catatonit \
        libpango-1.0-0 \
        libpangoft2-1.0-0 \
        libharfbuzz0b \
        libffi8 \
        libjpeg-turbo8 \
        libcairo2 \
        libgdk-pixbuf-2.0-0 \
        shared-mime-info \
        fontconfig \
        fonts-dejavu-core \
        fonts-liberation \
        procps \
    && rm -rf /var/lib/apt/lists/*

# Docker Compose sends oomScoreAdj 0. Nested Podman cannot write
# /proc/self/oom_score_adj, so crun must drop the field before create.
# One heredoc for the whole RUN: a <<'EOF' inside a continued shell RUN
# ends the Dockerfile instruction at the first terminator.
RUN <<'EOF'
set -eu
mv /usr/bin/crun /usr/bin/crun.real
cat > /usr/bin/crun <<'SCRIPT'
#!/bin/sh
bundle=""
prev=""
for arg in "$@"; do
  if [ "$prev" = "--bundle" ]; then
    bundle=$arg
  fi
  prev=$arg
done
if [ -n "$bundle" ] && [ -f "$bundle/config.json" ] && command -v node >/dev/null 2>&1; then
  node -e 'const fs=require("fs"); const p=process.argv[1]; const spec=JSON.parse(fs.readFileSync(p,"utf8")); if (spec.process) delete spec.process.oomScoreAdj; fs.writeFileSync(p, JSON.stringify(spec));' "$bundle/config.json" || true
fi
exec /usr/bin/crun.real "$@"
SCRIPT
chmod 755 /usr/bin/crun
mkdir -p /etc/containers/containers.conf.d /usr/local/bin
cat > /etc/containers/containers.conf.d/99-e2e.conf <<'CONF'
[engine]
cgroup_manager = "cgroupfs"
events_logger = "file"
CONF
ln -sfn /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose
EOF

# Same pin as apps/frontend/firestore-rules-tests. The rules deploy loads
# google-auth-library from this global install.
RUN npm install -g firebase-tools@15.31.0 \
    && npm cache clean --force

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

RUN uv python install 3.14 \
    && ln -sfn "$(uv python find 3.14)" /usr/local/bin/python3.14
