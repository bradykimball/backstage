FROM ghcr.io/acme/base:debian-12@sha256:0b7e5c2d9a41f86e3b1c7d02a9e4f5b6c8d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5

ARG NODE_VERSION=22.11.0

LABEL org.opencontainers.image.title="Node.js 22 (Debian slim)" \
      org.opencontainers.image.description="Hardened Node.js 22 LTS runtime on Debian 12 slim" \
      io.acme.golden.lifecycle="production" \
      io.acme.golden.tier="supported" \
      io.acme.golden.eol="2027-04-30" \
      io.acme.golden.tags="node,debian,runtime" \
      io.acme.golden.image-ref="ghcr.io/acme/node:22"

USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl xz-utils \
 && curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz \
    | tar -xJ -C /usr/local --strip-components=1 \
 && apt-get purge -y curl xz-utils && rm -rf /var/lib/apt/lists/*
USER 10001
