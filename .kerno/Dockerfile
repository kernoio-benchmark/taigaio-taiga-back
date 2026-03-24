# Taiga Backend Dockerfile
# Multi-stage build for production deployment

# Build stage
FROM python:3.10.15-slim-bookworm AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    zlib1g-dev \
    libtiff-dev \
    libwebp-dev \
    libopenjp2-7-dev \
    && rm -rf /var/lib/apt/lists/*

# Create virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy requirements file
COPY requirements.txt /tmp/requirements.txt

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir -r /tmp/requirements.txt

# Final stage
FROM python:3.10.15-slim-bookworm

# Install runtime dependencies and curl for healthchecks
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    libpq5 \
    libjpeg62-turbo \
    libpng16-16 \
    libfreetype6 \
    zlib1g \
    libtiff6 \
    libwebp7 \
    libopenjp2-7 \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN groupadd -r taiga && useradd -r -g taiga -d /home/taiga -m taiga

# Copy virtual environment from builder
COPY --chown=taiga:taiga --from=builder /opt/venv /opt/venv

# Set environment variables
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Set working directory
WORKDIR /home/taiga/app

# Copy application code
COPY --chown=taiga:taiga . .

# Switch to non-root user
USER taiga

# Expose port for Gunicorn
EXPOSE 8000

# Default command - run Gunicorn
CMD ["gunicorn", "taiga.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "sync"]
