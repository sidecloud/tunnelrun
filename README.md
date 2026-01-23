# Tunnelrun

This solutions allows you to run a container that automatically binds to a CloudFlareTunnel, leveraging Zero Trust Access and making the access to your LLMs secure.

The current LLM solutions supported are:

* Ollama;
* LLama-Server.

## Features

- **Ollama Integration**: Runs Ollama for serving large language models
- **Llama-server Support**: Integrated support for llama-server as an LLM backend
- **Cloudflare Tunnel**: Securely exposes LLM services to the internet without opening ports
- **Supervisor Management**: Uses supervisord to manage LLM and Cloudflare Tunnel processes

## Prerequisites

To be able to fully test this repository, you'll need to have the following tools installed:

- Docker and Docker Compose installed

Additionally,since the CloudFlare tunnel integration you also need:

- Cloudflare account with tunnel configured.
- Cloudflare Tunnel token (TUNNEL_TOKEN)

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/sidecloud/tunnelrun.git
cd tunnelrun
```

### 2. Set up environment variables

Create a `.env` file with your Cloudflare Tunnel token:

```bash
echo "TUNNEL_TOKEN=your_cloudflare_tunnel_token_here" > .env
```

### 3. Choose what container will be started

```bash
docker compose scale ollama=1 # To create an ollama container using the github already built image.
# OR
docker compose scale llama-server=1 # To create a llama-server container using the github already built image.
```

> LLama-server normally requires a bit more config, check `https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md` for what environment varibles you can use or model-presets/router mode.

### 4. [OPTIONAL] Build the images locally

```bash
docker compose build
```

### 4. On CloudFlare

Considering that you are using a token-based mode configuration, were the configuration stay in the cloudflare, make sure that:

* The configuration points to `http://localhost:11434` to expose ollama;
* The configuration points to ``http://localhost:8080` to expose llama-server.

## Configuration

### Environment Variables

The following environment variable is required for cloudflare token based mode:

- `TUNNEL_TOKEN` (required): Your Cloudflare Tunnel authentication token

> Both OLLAMA and LLAMA-SERVER can be configured using environment variables, so change the environments accordingly in the docker-compose.yaml file.

### Ports

- Ollama serves on port 11434 within the container
- Cloudflare Tunnel handles external connectivity

## Architecture

Each container run a LLM backend services managed by supervisord:

1. **LLM Backend**: Serves LLM models port 11434 (Ollama) or 8080 (llama-server);
2. **Cloudflare Tunnel**: Creates a secure tunnel to expose the LLM service externally.

## Customization

### Modifying Ollama Version

Edit the `deploy/Dockerfile` and change the base image tag:

```dockerfile
FROM ollama/ollama:0.14.3 AS ollama
```

### Modifying Cloudflare Tunnel Version

Update the cloudflared download URL in the Dockerfile:

```dockerfile
wget https://github.com/cloudflare/cloudflared/releases/download/2025.11.1/cloudflared-linux-$ARCH
```

## License

This project is open source and available under the MIT License.

## Support

For issues or questions, please open an issue on the GitHub repository.
