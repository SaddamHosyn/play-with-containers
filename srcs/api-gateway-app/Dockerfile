FROM debian:bullseye

# Install Python 3.9 and pip with minimal dependencies
RUN apt-get update && apt-get install -y \
    python3.9 \
    python3-pip \
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
HEALTHCHECK --interval=15s --timeout=5s --retries=5 --start-period=20s \
    CMD python3 -c "import requests; requests.get('http://localhost:3000/')" || exit 1

# Run application
ENTRYPOINT ["python3", "server.py"]
