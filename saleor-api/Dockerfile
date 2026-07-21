# Use Python 3.12 in a smaller base image
FROM python:3.12-slim

# Install the Linux packages required by Saleor
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    gettext \
    libffi8 \
    libgdk-pixbuf-xlib-2.0-0 \
    liblcms2-2 \
    libopenjp2-7 \
    libssl3 \
    libtiff6 \
    libwebp7 \
    shared-mime-info \
    media-types \
    libpq5 \
    libmagic1 \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies directly into the container
ENV POETRY_VERSION=1.8.4 \
    POETRY_VIRTUALENVS_CREATE=false

RUN pip install --no-cache-dir "poetry==$POETRY_VERSION" \
    && poetry config installer.max-workers 1

# Copy dependency files separately so Docker can cache this step
COPY pyproject.toml poetry.lock ./
RUN poetry install --no-root

# Copy the Saleor application
COPY . .

# Create the non-root user required by the assignment
RUN useradd -U saleor \
    && chown -R saleor:saleor /app

# Collect Django static files
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/static/}

RUN SECRET_KEY=dummy \
    STATIC_URL=${STATIC_URL} \
    python3 manage.py collectstatic --no-input

# Run the application as a non-root user
USER saleor

EXPOSE 8000

CMD ["sh", "-c", "until python manage.py migrate --noinput; do echo 'Waiting for PostgreSQL...'; sleep 5; done; exec gunicorn --bind :8000 --workers 4 --worker-class saleor.asgi.gunicorn_worker.UvicornWorker saleor.asgi:application"]