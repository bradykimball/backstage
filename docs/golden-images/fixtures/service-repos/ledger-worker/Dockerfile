# Expect: base-image python-3-12 from the FINAL stage only.
# The builder stage uses node-22 to build assets; that must NOT create a relation.
FROM ghcr.io/acme/node:22-2026.10.27@sha256:7d6c5b4a3f2e1d0c9b8a7f6e5d4c3b2a1f0e9d8c7b6a5f4e3d2c1b0a9f8e7d6c AS assets
WORKDIR /src
COPY web/ .
RUN npm ci && npm run build

FROM ghcr.io/acme/python:3.12-2026.10.16@sha256:1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b
WORKDIR /app
COPY --from=assets /src/dist ./static
COPY . .
CMD ["python", "-m", "ledger"]
