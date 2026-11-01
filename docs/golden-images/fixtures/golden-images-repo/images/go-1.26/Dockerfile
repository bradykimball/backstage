# Multi-stage: the provider must read labels and FROM from the FINAL stage only.
# The final base is outside the golden images, so: no dependsOn.
# runtime-version comes from the builder stage's GO_VERSION (no *_VERSION ARG in the final stage).
FROM ghcr.io/acme/base:debian-12@sha256:0b7e5c2d9a41f86e3b1c7d02a9e4f5b6c8d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5 AS builder
ARG GO_VERSION=1.26.2
LABEL io.acme.golden.tier="best-effort"
RUN mkdir -p /out/etc && cp -r /etc/ssl /out/etc/ && cp -r /usr/share/zoneinfo /out/zoneinfo

FROM gcr.io/distroless/static-debian12:nonroot@sha256:6ec5aa99dc335666e79dc64e4a6c8b89c33a543a1967f20d360922a80dd21f02
COPY --from=builder /out/etc/ssl /etc/ssl
COPY --from=builder /out/zoneinfo /usr/share/zoneinfo

LABEL org.opencontainers.image.title="Go 1.26 runtime (distroless static)" \
      org.opencontainers.image.description="Distroless static runtime for Go 1.26 binaries, with CA certs and tzdata" \
      io.acme.golden.lifecycle="production" \
      io.acme.golden.tier="supported" \
      io.acme.golden.eol="2027-02-28" \
      io.acme.golden.tags="go,distroless,runtime" \
      io.acme.golden.image-ref="ghcr.io/acme/go:1.26"
