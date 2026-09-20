# Tunnelrun

This solutions allows you to run a container that automatically binds to a CloudFlareTunnel, leveraging Zero Trust Access and making the access to your LLMs secure.

The current LLM solutions supported are:

* Ollama;
* LLama-Server (CUDA);
* LLama-Server (Vulkan);
* LLama-Server (ROCm);
* vLLM (CUDA);
* vLLM (ROCm — local AMD GPU testing).
* vLLM Diffusion (CUDA — pinned vLLM source build);
* vLLM Diffusion (ROCm — pinned vLLM source build).

## Features

- **Ollama Integration**: Runs Ollama for serving large language models
- **Llama-server Support**: Integrated CUDA, Vulkan, and ROCm support for llama-server as an LLM backend
- **vLLM Support**: Integrated support for vLLM as an OpenAI-compatible backend (CUDA and ROCm)
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
docker compose scale ollama=1       # Ollama
# OR
docker compose scale llama-server=1 # llama-server
# OR
docker compose scale llama-server-vulkan=1 # llama-server (Vulkan)
# OR
docker compose scale llama-server-rocm=1   # llama-server (ROCm)
# OR
docker compose scale vllm=1         # vLLM (CUDA)
# OR
docker compose scale vllm-rocm=1    # vLLM (ROCm — AMD GPU)
# OR
docker compose scale vllm-diffusion=1      # vLLM Diffusion (CUDA)
# OR
docker compose scale vllm-rocm-diffusion=1 # vLLM Diffusion (ROCm)
```

> LLama-server normally requires a bit more config, check `https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md` for what environment varibles you can use or model-presets/router mode.

> The Vulkan service passes `/dev/dri` through to the container and requires a host with a working Vulkan driver.

> The ROCm service passes `/dev/kfd` and `/dev/dri` through to the container and requires a ROCm-compatible AMD GPU and host driver.

> vLLM requires a configuration file — see the [vLLM Configuration](#vllm-configuration) section below.

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

### vLLM Configuration

vLLM requires a YAML config file mounted at `/etc/vllm/config.yaml` inside the container. A default config is provided at `deploy/vllm.config.yaml`.

The config file maps directly to `vllm serve` CLI arguments:

```yaml
model: cyankiwi/Qwen3.5-9B-AWQ-4bit
served_model_name:
  - qwen35-9b
host: 0.0.0.0
port: 8080
max_model_len: 262144
enable_auto_tool_choice: true
tool_call_parser: qwen3_coder
# ... any other vllm serve flags
```

Full list of supported options: `vllm serve --help=all` or https://docs.vllm.ai/en/latest/configuration/serve_args.html

To use a different model, edit `deploy/vllm.config.yaml` before starting the container. No environment variables are required for the model — everything is driven by the config file.

### Pinned vLLM Diffusion Builds

`vllm-diffusion` and `vllm-rocm-diffusion` are built from the vLLM source
commit in `versions/diffusion_version.txt`, rather than from a released
`vllm-openai` image. Update that file manually to change the source revision.

For a local build, export the same values used by CI:

```bash
export VLLM_DIFFUSION_COMMIT="$(cat versions/diffusion_version.txt)"
export VLLM_DIFFUSION_COMMIT_SHORT="${VLLM_DIFFUSION_COMMIT:0:7}"
docker compose build vllm-diffusion vllm-rocm-diffusion
```

Published images have a moving alias and a commit-specific tag, for example
`sidecloud8808/tunnelrun:vllm-diffusion` and
`sidecloud8808/tunnelrun:vllm-diffusion-ceb8eeb`.

The source-built images are published only by manually running the
`Docker Diffusion Images` GitHub Actions workflow. It can build both variants
or only CUDA/ROCm.

> These source builds are substantially heavier than the standard vLLM wrapper
> images. They require network access to the pinned vLLM repository and may
> exceed normal GitHub-hosted runner disk or time limits.

### Ports

- Ollama serves on port 11434 within the container
- Cloudflare Tunnel handles external connectivity

## Architecture

Each container run a LLM backend services managed by supervisord:

1. **LLM Backend**: Serves LLM models on port 11434 (Ollama) or 8080 (llama-server, vLLM);
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
