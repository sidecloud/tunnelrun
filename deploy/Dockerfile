# syntax=docker/dockerfile:1

ARG OLLAMA_TAG
ARG LLAMA_SERVER_TAG

FROM busybox AS downloader
WORKDIR /usr/src/
ARG TARGETPLATFORM
RUN export ARCH=$(echo $TARGETPLATFORM | cut -d "/" -f2) && \
    wget https://github.com/cloudflare/cloudflared/releases/download/2025.11.1/cloudflared-linux-$ARCH -O cloudflared
RUN chmod +x cloudflared

# Ollama target
FROM ollama/ollama:${OLLAMA_TAG} AS ollama
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
COPY --from=downloader /usr/src/cloudflared /usr/local/bin/cloudflared
RUN uv python install 3.12 --default && ln -s /root/.local/bin/python /usr/local/bin/python
RUN uv tool install supervisor
COPY ollama_supervisord.conf /etc/supervisor/conf.d/ollama.conf
VOLUME /home/ollama/.ollama
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]

# llama.cpp target
FROM ghcr.io/ggml-org/llama.cpp:server-cuda-${LLAMA_SERVER_TAG} AS llama-server
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
COPY --from=downloader /usr/src/cloudflared /usr/local/bin/cloudflared
RUN uv python install 3.12 --default && ln -s /root/.local/bin/python /usr/local/bin/python
RUN uv tool install supervisor
COPY llama_server_supervisord.conf /etc/supervisor/conf.d/llama-server.conf
VOLUME /root/.cache/llama.cpp
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
