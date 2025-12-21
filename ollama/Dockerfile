# syntax=docker/dockerfile:1
FROM busybox AS downloader
WORKDIR /usr/src/
ARG TARGETPLATFORM
RUN export ARCH=$(echo $TARGETPLATFORM | cut -d "/" -f2) && \
    wget https://github.com/cloudflare/cloudflared/releases/download/2025.11.1/cloudflared-linux-$ARCH -O cloudflared
RUN chmod +x cloudflared

FROM ollama/ollama:0.13.5 AS ollama
ENV DEBIAN_FRONTEND=noninteractive UV_NO_CACHE=1 UV_TOOL_BIN_DIR=/usr/local/bin PYTHONUNBUFFERED=1
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/
COPY --from=downloader /usr/src/cloudflared /usr/local/bin/cloudflared
COPY supervisord.conf /etc/supervisor/supervisord.conf
RUN uv python install 3.12 --default && ln -s /root/.local/bin/python /usr/local/bin/python
RUN uv tool install supervisor

VOLUME /home/ollama/.ollama
ENTRYPOINT ["/usr/local/bin/supervisord"]
CMD ["-c", "/etc/supervisor/supervisord.conf"]
