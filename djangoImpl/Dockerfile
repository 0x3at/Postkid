# syntax=docker/dockerfile:1

ARG PYTHON_VERSION=3.13.5
FROM python:${PYTHON_VERSION}-slim as base

# Install uv
RUN pip install uv

WORKDIR /app

# Create a non-privileged user that the app will run under.
# See https://docs.docker.com/go/dockerfile-user-best-practices/
ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

# Create a cache directory for uv
RUN mkdir -p /app/.cache/uv

# Change ownership of the cache directory to appuser
RUN chown -R appuser:appuser /app/.cache/uv

# Change ownership of /app to appuser
RUN chown -R appuser:appuser /app

# Switch to the non-privileged user to run the application.
USER appuser

# Set the UV_CACHE_DIR environment variable
ENV UV_CACHE_DIR=/app/.cache/uv

# Set up virtual environment with uv
RUN uv venv .venv
ENV VIRTUAL_ENV="/app/.venv"
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Set the UV_CACHE_DIR environment variable
ENV UV_CACHE_DIR=/app/.cache/uv

# Copy requirements file
COPY pyproject.toml uv.lock ./

# Install dependencies using uv
RUN uv pip install --no-cache-dir .

# Copy the source code into the container.
COPY . .

# Expose the port that the application listens on.
EXPOSE 8118

# Run the application.
CMD uv run Postkid/manage.py runserver 0.0.0.0:8118
