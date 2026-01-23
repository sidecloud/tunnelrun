# syntax=docker/dockerfile:1

ARG OLLAMA_TAG
ARG LLAMA_SERVER_TAG
ARG CLOUDFLARED_TAG

FROM cloudflare/cloudflared:${CLOUDFLARED_TAG} as cloudflared
# Ollama target
FROM ollama/ollama:${OLLAMA_TAG} AS ollama
ENV UV_TOOL_BIN_DIR=/usr/local/bin
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
COPY --from=cloudflared /usr/local/bin/cloudflared /usr/local/bin/cloudflared
RUN uv python install 3.12 --default && ln -s /root/.local/bin/python /usr/local/bin/python
RUN uv tool install supervisor
COPY ollama_supervisord.conf /etc/supervisor/supervisord.conf
VOLUME /home/ollama/.ollama
ENTRYPOINT ["/usr/local/bin/supervisord"]
CMD ["-c", "/etc/supervisor/supervisord.conf"]

# llama.cpp target
FROM ghcr.io/ggml-org/llama.cpp:server-cuda-${LLAMA_SERVER_TAG} AS llama-server
ENV UV_TOOL_BIN_DIR=/usr/local/bin
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
COPY --from=cloudflared /usr/local/bin/cloudflared /usr/local/bin/cloudflared
RUN uv python install 3.12 --default && ln -s /root/.local/bin/python /usr/local/bin/python
RUN uv tool install supervisor
COPY llama_server_supervisord.conf /etc/supervisor/supervisord.conf
VOLUME /root/.cache/llama.cpp
ENTRYPOINT ["/usr/local/bin/supervisord"]
CMD ["-c", "/etc/supervisor/supervisord.conf"]
