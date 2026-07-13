FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/app/.venv/bin:$PATH"

RUN pip install \
    --no-cache-dir \
    uv

COPY pyproject.toml uv.lock ./

RUN uv sync \
    --frozen \
    --no-dev

COPY . .

EXPOSE 8000

CMD [
    "fastapi",
    "run",
    "main.py",
    "--host",
    "0.0.0.0",
    "--port",
    "8000"
]