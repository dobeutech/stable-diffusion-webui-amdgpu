FROM mcr.microsoft.com/devcontainers/python:3.10-bookworm

RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends \
        libgl1 \
        libglib2.0-0 \
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
