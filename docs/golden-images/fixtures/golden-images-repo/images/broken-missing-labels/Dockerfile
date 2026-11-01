# Must still be emitted, with metadata-incomplete: "true" and metadata-errors naming:
#   - missing io.acme.golden.tier
#   - missing io.acme.golden.eol
#   - invalid io.acme.golden.tags (uppercase and spaces)
#   - no image reference (no image-ref label, no goldenImages.imageRefs entry)
FROM debian:12-slim@sha256:3f1d6c17773a45c97bd8f158d665c9709d7b29ed7917ac934086ad96f92e4510

LABEL org.opencontainers.image.title="Half-finished image" \
      org.opencontainers.image.description="Someone copied a Dockerfile and forgot the labels" \
      io.acme.golden.lifecycle="experimental" \
      io.acme.golden.tags="Python, Debian"
