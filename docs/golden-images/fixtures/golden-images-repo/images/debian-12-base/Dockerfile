# Root image: FROM points outside the golden images, so no dependsOn.
FROM debian:12-slim@sha256:3f1d6c17773a45c97bd8f158d665c9709d7b29ed7917ac934086ad96f92e4510

LABEL org.opencontainers.image.title="Debian 12 base (hardened)" \
      org.opencontainers.image.description="Hardened Debian 12 slim base for all Debian golden images" \
      io.acme.golden.lifecycle="production" \
      io.acme.golden.tier="supported" \
      io.acme.golden.eol="2028-06-30" \
      io.acme.golden.tags="base,debian" \
      io.acme.golden.image-ref="ghcr.io/acme/base:debian-12"

RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates tzdata libxml2 \
 && rm -rf /var/lib/apt/lists/* \
 && useradd --uid 10001 --create-home app

USER 10001
