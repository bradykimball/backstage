# Expect: base-image component:default/python-3-12, pinned 3.12-2026.09.15 (two files reference it).
FROM ghcr.io/acme/python:3.12-2026.09.15@sha256:4c88e2a1b0f9d8c7e6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c8b7a6f5b19a
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "-m", "payments"]
