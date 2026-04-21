FROM debian:bullseye

# Install Python 3.9, pip, and curl
RUN apt-get update && apt-get install -y \
    python3.9 \
    python3-pip \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements first for better layer caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=10s --timeout=5s --retries=5 --start-period=20s \
    CMD curl -f http://localhost:3000/health || exit 1

# Run application
ENTRYPOINT ["python3", "server.py"]
