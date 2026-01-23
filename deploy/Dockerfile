# syntax=docker/dockerfile:1
FROM busybox AS downloader
WORKDIR /usr/src/
ARG TARGETPLATFORM
RUN export ARCH=$(echo $TARGETPLATFORM | cut -d "/" -f2) && \
    wget https://github.com/cloudflare/cloudflared/releases/download/2025.11.1/cloudflared-linux-$ARCH -O cloudflared
RUN chmod +x cloudflared

# Base stage for common dependencies
FROM ubuntu:22.04 AS base
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        supervisor \
        && rm -rf /var/lib/apt/lists/*
RUN mkdir -p /etc/supervisor/conf.d

# Ollama target
FROM ollama/ollama:0.14.3 AS ollama
COPY --from=base /usr/bin/supervisord /usr/bin/supervisord
COPY --from=base /etc/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY --from=downloader /usr/src/cloudflared /usr/local/bin/cloudflared
COPY ollama_supervisord.conf /etc/supervisor/conf.d/ollama.conf
RUN uv python install 3.12 --default && ln -s /root/.local/bin/python /usr/local/bin/python
RUN uv tool install supervisor
VOLUME /home/ollama/.ollama
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]

# llama.cpp target
FROM ghcr.io/ggml-org/llama.cpp:server-cuda-b7786 AS llama-server
COPY --from=base /usr/bin/supervisord /usr/bin/supervisord
COPY --from=base /etc/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY --from=downloader /usr/src/cloudflared /usr/local/bin/cloudflared
COPY llama_server_supervisord.conf /etc/supervisor/conf.d/llama-server.conf
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
