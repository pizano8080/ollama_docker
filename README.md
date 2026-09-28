# Ollama + Open WebUI + Kokoro Docker

This Docker Compose project runs **Ollama**, **Open WebUI**, and **Kokoro TTS** together with NVIDIA GPU support.

The setup uses Docker Compose with persistent named volumes so application data survives container recreation or image updates.

## Components

* **Ollama** — Local LLM inference using the NVIDIA GPU.
* **Open WebUI** — Web interface for interacting with Ollama models.
* **Kokoro** — FastAPI-based text-to-speech service using the NVIDIA GPU.
* **Docker Compose** — Manages the three services and their networking.

## Architecture

```text
                    Docker Compose
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
      Ollama          Open WebUI        Kokoro
      :11434            :3030           :8880
          |               |               |
          +-------+-------+               |
                  |                       |
             Ollama Models            TTS Models
                  |                       |
            ollama volume              Container
```

## Persistent Storage

Docker named volumes are used for data that needs to survive container recreation.

### Ollama

```yaml
- ollama:/root/.ollama
```

The `ollama` volume stores the downloaded Ollama models.

Removing and recreating the Ollama container does **not** remove the models as long as the Docker volume is retained.

### Open WebUI

```yaml
- webui-data:/app/backend/data
```

The `webui-data` volume stores Open WebUI's persistent application data, including its configuration and user data.

## GPU Support

Both Ollama and Kokoro are configured to use the NVIDIA GPU:

```yaml
gpus: all
```

This requires a working NVIDIA driver and Docker NVIDIA GPU support on the host.

Ollama uses the GPU for LLM inference, while Kokoro uses the GPU for text-to-speech generation.

## Ollama

The Ollama container uses the official image:

```yaml
image: ollama/ollama:latest
```

Ollama is exposed on:

```text
11434
```

The host port is mapped directly to the container:

```yaml
ports:
  - "11434:11434"
```

Ollama model data is stored in the persistent `ollama` Docker volume.

## Open WebUI

Open WebUI is built from:

```yaml
ghcr.io/open-webui/open-webui:ollama
```

The WebUI is exposed on host port:

```text
3030
```

and maps to port `8080` inside the container:

```yaml
ports:
  - 3030:8080
```

Open WebUI connects to Ollama through:

```text
http://localhost:11434
```

The WebUI data is stored in the persistent `webui-data` Docker volume.

## Kokoro

Kokoro uses the GPU-enabled image:

```yaml
image: ghcr.io/remsky/kokoro-fastapi-gpu:latest
```

The service is exposed on:

```text
8880
```

The container is configured to use the GPU:

```yaml
environment:
  - USE_GPU=true
```

and:

```yaml
gpus: all
```

### CPU Version

A CPU version is also included in the Compose file as a commented-out option:

```yaml
#image: ghcr.io/remsky/kokoro-fastapi-cpu:latest
```

To switch to the CPU version, replace the GPU image with:

```yaml
image: ghcr.io/remsky/kokoro-fastapi-cpu:latest
```

and remove or disable the GPU-specific configuration:

```yaml
gpus: all
```

and:

```yaml
USE_GPU=true
```

## Docker Volumes

The Compose file creates two persistent named volumes:

```yaml
volumes:
  ollama: {}
  webui-data: {}
```

These are intentionally separate from the containers.

This allows the containers to be deleted and recreated without losing the downloaded Ollama models or Open WebUI data.

## Ports

| Service    | Host Port | Container Port | Purpose        |
| ---------- | --------: | -------------: | -------------- |
| Ollama     |     11434 |          11434 | Ollama API     |
| Open WebUI |      3030 |           8080 | Web interface  |
| Kokoro     |      8880 |           8880 | Kokoro FastAPI |

## Starting the Stack

From the directory containing `docker-compose.yml`:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

## Stopping the Stack

```bash
docker compose down
```

`docker compose down` removes the containers but leaves the named Docker volumes intact.

## Updating the Containers

Pull the latest images:

```bash
docker compose pull
```

Then recreate the containers:

```bash
docker compose up -d
```

The persistent volumes remain attached, so Ollama models and Open WebUI data are retained.

## Checking GPU Access

Check that Docker can see the NVIDIA GPU:

```bash
docker run --rm --gpus all nvidia/cuda:13.0.0-base-ubuntu24.04 nvidia-smi
```

The Ollama and Kokoro containers should also be able to access the GPU when running with:

```yaml
gpus: all
```

## Project Design

The Compose file is intentionally kept simple:

* Docker Compose manages the services.
* Ollama stores models in a persistent Docker volume.
* Open WebUI stores its application data in a persistent Docker volume.
* Ollama and Kokoro have access to all available NVIDIA GPUs.
* Services can be recreated without losing persistent application data.
* Container images can be updated independently from the persistent data.

## File Structure

A basic project layout is:

```text
ollama_docker_gpu/
├── docker-compose.yml
└── README.md
```

Docker manages the persistent application data through named volumes rather than storing it directly in the project directory.
