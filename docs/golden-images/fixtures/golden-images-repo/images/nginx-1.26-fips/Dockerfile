# FROM uses an ARG: the provider must resolve ${BASE_IMAGE} from its default
# and still find debian-12-base as the base.
# CODEOWNERS lists @acme/platform-edge first; it isn't in ownerMap, so owner falls back to defaultOwner.
ARG BASE_IMAGE=ghcr.io/acme/base:debian-12
FROM --platform=linux/amd64 ${BASE_IMAGE}

ARG NGINX_VERSION=1.26.2

LABEL org.opencontainers.image.title="nginx 1.26 (FIPS)" \
      org.opencontainers.image.description="nginx with OpenSSL in FIPS mode, for edge proxies" \
      io.acme.golden.lifecycle="production" \
      io.acme.golden.tier="best-effort" \
      io.acme.golden.eol="2027-05-31" \
      io.acme.golden.tags="proxy,fips" \
      io.acme.golden.image-ref="ghcr.io/acme/nginx:1.26-fips"

USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends nginx=${NGINX_VERSION}* \
 && rm -rf /var/lib/apt/lists/*
USER 10001
