# =====================================================
# Stage 1: Build Arguments
# =====================================================

ARG PYTHON_VERSION=3.11
ARG APP_PORT=8001
ARG APP_NAME=booktrack-auth
ARG APP_VERSION=1.0.0

# =====================================================
# Stage 2: Build Dependencies
# =====================================================

FROM python:${PYTHON_VERSION}-slim-bookworm AS builder

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /build

COPY requirements.txt .

RUN pip install \
    --no-cache-dir \
    --prefix=/install \
    -r requirements.txt

# =====================================================
# Stage 3: Secure Runtime
# =====================================================

FROM gcr.io/distroless/python3-debian12:nonroot

ARG APP_PORT=8001
ARG APP_NAME=booktrack-auth
ARG APP_VERSION=1.0.0

# =====================================================
# Application Metadata
# =====================================================

LABEL application="${APP_NAME}" \
      version="${APP_VERSION}" \
      security="hardened"

# =====================================================
# Environment Variables
# =====================================================

ENV APP_NAME=${APP_NAME} \
    APP_VERSION=${APP_VERSION} \
    APP_ENV=production \
    APP_PORT=${APP_PORT} \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/install/lib/python3.11/site-packages \
    HOME=/tmp \
    TMPDIR=/tmp

# =====================================================
# Working Directory
# =====================================================

WORKDIR /app

# =====================================================
# Copy Python Dependencies
# =====================================================

COPY --from=builder /install /install

# =====================================================
# Copy Application
# =====================================================

COPY --chown=65532:65532 . /app

# =====================================================
# Non-root User
# =====================================================

USER 65532:65532

# =====================================================
# Application Port
# =====================================================

EXPOSE ${APP_PORT}

# =====================================================
# Health Check
# =====================================================

HEALTHCHECK --interval=30s \
            --timeout=5s \
            --start-period=30s \
            --retries=3 \
            CMD ["python3", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8001/health', timeout=3)"]

# =====================================================
# Application Startup
# =====================================================

CMD ["-c", "import os,uvicorn; uvicorn.run('main:app',host='0.0.0.0',port=int(os.environ.get('APP_PORT','8001')))"]