# Python 3.12 golden image

Its presence makes the provider set `backstage.io/techdocs-ref` for this image.

## What's included
- Python 3.12, pip, venv
- CA certificates, tzdata
- Non-root user `app` (uid 10001)

## Not for
- Data-science workloads. Use `python-3.12-ml` when it exists.
