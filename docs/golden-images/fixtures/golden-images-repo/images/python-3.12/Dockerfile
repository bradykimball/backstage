# Snapshot fixture: must produce fixtures/expected/python-3-12.entity.json exactly.
FROM ghcr.io/acme/base:debian-12@sha256:0b7e5c2d9a41f86e3b1c7d02a9e4f5b6c8d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5

ARG PYTHON_VERSION=3.12.7

LABEL org.opencontainers.image.title="Python 3.12 (Debian slim)" \
      org.opencontainers.image.description="Hardened Python 3.12 runtime on Debian 12 slim. Non-root, pip and venv included." \
      io.acme.golden.lifecycle="production" \
      io.acme.golden.tier="supported" \
      io.acme.golden.eol="2028-10-31" \
      io.acme.golden.tags="python,debian,runtime,non-root" \
      io.acme.golden.image-ref="ghcr.io/acme/python:3.12"

USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      python3.12=${PYTHON_VERSION}* python3-pip python3-venv \
      libexpat1 libsqlite3-0 liblzma5 \
 && rm -rf /var/lib/apt/lists/*

USER 10001
WORKDIR /app
