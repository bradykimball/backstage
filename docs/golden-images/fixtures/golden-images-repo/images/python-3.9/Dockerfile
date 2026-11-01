# Deprecated image with a successor. Its base (debian-11) is not a golden image in this repo,
# so it has no dependsOn.
FROM ghcr.io/acme/base:debian-11@sha256:9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b

ARG PYTHON_VERSION=3.9.20

LABEL org.opencontainers.image.title="Python 3.9 (Debian 11)" \
      org.opencontainers.image.description="Deprecated. Migrate to python-3.12." \
      io.acme.golden.lifecycle="deprecated" \
      io.acme.golden.tier="best-effort" \
      io.acme.golden.eol="2025-10-31" \
      io.acme.golden.tags="python,debian" \
      io.acme.golden.successor="python-3-12" \
      io.acme.golden.image-ref="ghcr.io/acme/python:3.9"

USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends python3.9=${PYTHON_VERSION}* \
 && rm -rf /var/lib/apt/lists/*
USER 10001
