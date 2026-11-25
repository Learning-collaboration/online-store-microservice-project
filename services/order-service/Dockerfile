# ==========================================
# Stage 1: Build & Dependencies
# ==========================================
FROM python:3.12-slim AS builder

# Install system build tools and PostgreSQL development headers
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy dependency manifest
COPY requirements.txt ./

# Install dependencies into an isolated prefix directory using --prefix=/install
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Copy application source code
COPY . /app

# ==========================================
# Stage 2: Production Runtime
# ==========================================
FROM python:3.12-slim

# Install only the lightweight runtime PostgreSQL client library (`libpq5`)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 1. Copy the prefix install directly into /usr/local (merging bin and site-packages natively)
COPY --from=builder /install /usr/local

# 2. Copy ONLY application code (requirements.txt is left behind)
COPY --from=builder /app/app ./app

# Create a secure non-root user and assign permissions
RUN useradd -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Expose service port 8084
EXPOSE 8084

# Uvicorn is now natively available in PATH because it was merged into /usr/local/bin
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8084", "--workers", "1"]