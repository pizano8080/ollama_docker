# Ollama + Open WebUI + Kokoro Docker

This Docker Compose project runs **Ollama**, **Open WebUI**, and **Kokoro TTS** together.

The setup uses Docker Compose with persistent named volumes so application data survives container recreation or image updates.

## Components

* **Ollama** — Local LLM inference.
* **Open WebUI** — Web interface for interacting with Ollama models.
* **Kokoro** — FastAPI-based text-to-speech service.
* **Docker Compose** — Manages the services and networking.

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
             Ollama Models            TTS Service
                  |
            ollama volume
```

## Quick Installation

1. Clone the repository:

```bash
git clone https://github.com/pizano8080/ollama_docker.git
```

2. Change to the project directory:

```bash
cd ollama_docker
```

3. Update `.env` if needed:

```env
OLLAMA_API_KEY=your_ollama_api_key_here
OPENAI_API_KEY=your_openai_api_key_here
WEBUI_SECRET_KEY=your_random_secret_here
```

**Important:** The `.env` file contains API keys and secrets. Secure the file after updating it and do not share or publish it with real credentials.

4. Start the containers:

```bash
docker compose up -d
```

5. If using Ollama Cloud models, sign in:

```bash
docker exec -it ollama ollama signin
```

Open the URL provided by Ollama and authorize the Docker installation.

6. Open Open WebUI:

```text
http://localhost:3030
```

## Persistent Storage

Docker named volumes keep important data separate from the containers.

### Ollama

```yaml
- ollama:/root/.ollama
```

Stores downloaded Ollama models and Ollama authentication.

### Open WebUI

```yaml
- webui-data:/app/backend/data
```

Stores Open WebUI configuration and user data.

Deleting and recreating the containers does not remove these volumes.

## GPU Support

The Compose file is configured for NVIDIA GPU support:

```yaml
gpus: all
```

The Compose file can be modified for either **NVIDIA GPU** or **CPU-only** operation.

For CPU-only Kokoro, use:

```yaml
image: ghcr.io/remsky/kokoro-fastapi-cpu:latest
```

and remove:

```yaml
gpus: all
```

and:

```yaml
USE_GPU=true
```

## Ollama

Ollama uses the official image:

```yaml
image: ollama/ollama:latest
```

Ollama is available on port `11434`:

```yaml
ports:
  - "11434:11434"
```

### Ollama Cloud

Ollama Cloud models can be used after signing in:

```bash
docker exec -it ollama ollama signin
```

The Ollama authentication is stored in the persistent `ollama` volume.

### Ollama API Key

The API key can be supplied through `.env`:

```env
OLLAMA_API_KEY=your_ollama_api_key_here
```

and passed to the container through Docker Compose.

## Open WebUI

Open WebUI uses the published image:

```yaml
image: ghcr.io/open-webui/open-webui:ollama
```

The WebUI is available on port `3030`:

```yaml
ports:
  - 3030:8080
```

Open WebUI connects to Ollama through the Docker Compose network:

```text
http://ollama:11434
```

### WebUI Secret Key

The secret key is stored in `.env`:

```env
WEBUI_SECRET_KEY=your_random_secret_here
```

Keep the same value when recreating the container.

## OpenAI TTS

Open WebUI can use OpenAI for text-to-speech.

The Compose configuration uses:

```yaml
AUDIO_TTS_ENGINE=openai
AUDIO_TTS_OPENAI_API_BASE_URL=https://api.openai.com/v1
AUDIO_TTS_OPENAI_API_KEY=${OPENAI_API_KEY}
AUDIO_TTS_MODEL=tts-1
AUDIO_TTS_VOICE=alloy
```

The OpenAI API key is stored in `.env`:

```env
OPENAI_API_KEY=your_openai_api_key_here
```

## Kokoro

Kokoro uses the GPU-enabled image:

```yaml
image: ghcr.io/remsky/kokoro-fastapi-gpu:latest
```

The service is available on port `8880`:

```yaml
ports:
  - "8880:8880"
```

GPU operation is enabled with:

```yaml
USE_GPU=true
gpus: all
```

### CPU Version

For CPU-only operation, change the image to:

```yaml
image: ghcr.io/remsky/kokoro-fastapi-cpu:latest
```

and remove the GPU-specific settings.

## Environment File

Docker Compose automatically loads `.env` from the project directory.

Example:

```env
OLLAMA_API_KEY=your_ollama_api_key_here
OPENAI_API_KEY=your_openai_api_key_here
WEBUI_SECRET_KEY=your_random_secret_here
```

The values are referenced by `docker-compose.yml` using:

```yaml
${OLLAMA_API_KEY}
${OPENAI_API_KEY}
${WEBUI_SECRET_KEY}
```

Keep `.env` secured because it contains API keys and secrets.

## Docker Volumes

The Compose file creates two persistent named volumes:

```yaml
volumes:
  ollama: {}
  webui-data: {}
```

These volumes remain when the containers are removed.

## Ports

| Service    | Host Port | Container Port | Purpose       |
| ---------- | --------: | -------------: | ------------- |
| Ollama     |     11434 |          11434 | Ollama API    |
| Open WebUI |      3030 |           8080 | Web interface |
| Kokoro     |      8880 |           8880 | TTS API       |

## Starting the Stack

```bash
docker compose up -d
```

Check the containers:

```bash
docker compose ps
```

## Stopping the Stack

```bash
docker compose down
```

The containers are removed, but the persistent volumes remain.

## Updating the Containers

Pull the latest images:

```bash
docker compose pull
```

Recreate the containers:

```bash
docker compose up -d
```

Persistent volumes remain attached.

## Project Design

The Compose file is intentionally simple:

* Docker Compose manages the services.
* Ollama models are stored in a persistent Docker volume.
* Open WebUI data is stored in a persistent Docker volume.
* Ollama and Kokoro can use NVIDIA GPUs.
* The Compose file can be modified for CPU-only operation.
* Containers can be recreated without losing persistent data.

## File Structure

```text
ollama_docker/
├── docker-compose.yml
├── .env
└── README.md
```

Docker manages the persistent application data through named volumes rather than storing it directly in the project directory.
