# Python FastAPI App

## Overview

This folder contains a simple Python FastAPI application that returns a JSON response:

```json
{"status": "OK", "message": "Hello World"}
```

The app is packaged with Docker and exposed on port `8000`.

---

## Application Files

```text
python/
├── Dockerfile
├── app/
│   └── main.py
├── pyproject.toml
├── uv.lock
└── README.md
```

### Main application

The FastAPI app is defined in `app/main.py`.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"status": "OK", "message": "Hello World"}
```

---

## Dockerfile

The Dockerfile uses a multi-stage build approach with `uv` for dependency management and a slim Python runtime for the final image.

```dockerfile
FROM python:3.12-slim AS builder

COPY --from=astral/uv:latest /uv /uvx /bin/

WORKDIR /app

COPY ./pyproject.toml ./pyproject.toml
COPY ./uv.lock ./uv.lock

RUN uv sync --frozen --no-cache --no-install-project


FROM python:3.12-slim AS production

WORKDIR /app

COPY --from=builder /app/.venv /app/.venv

COPY app /app/app

ENV PATH="/app/.venv/bin:$PATH"
ENV PYTHONBUFFERED=1

EXPOSE 8000

CMD [ "fastapi" , "run" , "app/main.py" , "--host", "0.0.0.0", "--port" , "8000" ]
```

---

## Build the Image

```bash
cd assignment/docker-files-images/python
docker build -t python-app .
```

---

## Run the Container

```bash
docker run --name python-app-test -p 8000:8000 -d python-app
```

---

## Verify the Application

```bash
curl http://localhost:8000/
```

Expected output:

```json
{"status":"OK","message":"Hello World"}
```

You can also inspect logs:

```bash
docker logs python-app-test
```

---

## Notes

- Container port: `8000`
- Host port: `8000`
- Framework: FastAPI
- Purpose: return a simple "Hello World" response in JSON format

This app is part of the Docker assignment demonstrating how different runtimes and frameworks can be containerized and exposed with a consistent port mapping strategy.
