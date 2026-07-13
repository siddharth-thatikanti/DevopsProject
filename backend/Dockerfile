# =====================================
# Stage 1: Dependency Builder
# =====================================

FROM python:3.12-slim AS builder

WORKDIR /app

# Python and uv settings
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy

# Install uv
RUN pip install --no-cache-dir uv

# Copy dependency files first for Docker caching
COPY pyproject.toml uv.lock ./

# Install only production dependencies
# --no-install-project avoids installing the application
# before the source code is copied
RUN uv sync \
    --frozen \
    --no-dev \
    --no-install-project


# =====================================
# Stage 2: Production Runtime
# =====================================

FROM python:3.12-slim AS production

WORKDIR /app

# Production Python settings
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/app/.venv/bin:$PATH"

# Install curl only for the health check
RUN apt-get update && \
    apt-get install \
    -y \
    --no-install-recommends \
    curl && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
RUN addgroup \
    --system \
    appgroup && \
    adduser \
    --system \
    --ingroup appgroup \
    appuser

# Copy the virtual environment from builder
COPY --from=builder \
    --chown=appuser:appgroup \
    /app/.venv \
    /app/.venv

# Copy FastAPI application
COPY --chown=appuser:appgroup . .

# Run as a non-root user
USER appuser

EXPOSE 8000

# Check application health
HEALTHCHECK \
    --interval=30s \
    --timeout=5s \
    --start-period=10s \
    --retries=3 \
    CMD curl \
    --fail \
    http://localhost:8000/health \
    || exit 1

# Start FastAPI production server
CMD [
    "fastapi",
    "run",
    "main.py",
    "--host",
    "0.0.0.0",
    "--port",
    "8000"
]