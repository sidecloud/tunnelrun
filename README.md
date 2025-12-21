# Tunnelrun

A Docker-based solution for running LLM serving platforms with Cloudflare Tunnel integration. This project provides secure internet access to LLM serving solutions through Cloudflare's tunneling service.

## Features

- **Ollama Integration**: Runs Ollama 0.13.5 for serving large language models
- **Cloudflare Tunnel**: Securely exposes LLM services to the internet without opening ports
- **Multi-platform Support**: Builds for both ARM64 and AMD64 architectures
- **Supervisor Management**: Uses supervisord to manage LLM and Cloudflare Tunnel processes
- **Future Backend Support**: Planned support for vLLM and llama.cpp backends

## Prerequisites

- Docker and Docker Compose installed
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

### 3. Build and run the container

```bash
docker compose up --build
```

### 4. On CloudFlare

Make sure the tunnel is configured to expose ollama through `http://localhost:11434`.

## Configuration

### Environment Variables

- `TUNNEL_TOKEN` (required): Your Cloudflare Tunnel authentication token

### Ports

- Ollama serves on port 11434 within the container
- Cloudflare Tunnel handles external connectivity

## Architecture

The container runs two main services managed by supervisord:

1. **Ollama Service**: Serves LLM models on 0.0.0.0:11434
2. **Cloudflare Tunnel**: Creates a secure tunnel to expose the LLM service externally

Both services are configured with automatic restart policies and proper logging.

## Building

To build the Docker image manually:

```bash
docker compose build ollama
```

## Customization

### Modifying Ollama Version

Edit the `ollama/Dockerfile` and change the base image tag:

```dockerfile
FROM ollama/ollama:0.13.5 AS ollama
```

### Future Backend Support

Support for additional LLM backends (vLLM, llama.cpp) is planned for future releases. The project architecture is designed to accommodate multiple backend options.

### Modifying Cloudflare Tunnel Version

Update the cloudflared download URL in the Dockerfile:

```dockerfile
wget https://github.com/cloudflare/cloudflared/releases/download/2025.11.1/cloudflared-linux-$ARCH
```

## License

This project is open source and available under the MIT License.

## Support

For issues or questions, please open an issue on the GitHub repository.
