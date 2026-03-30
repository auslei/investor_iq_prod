# Investor IQ Prod

This repository runs the Investor IQ application stack from prebuilt Docker images. It is the deployment-oriented companion to the source repository in `../investor_iq` and is intended for starting the services, mounting project data, and using the UI and API.

## Stack

`docker-compose.yml` starts four containers:

- `qdrant`: vector database on ports `6333` and `6334`
- `extraction-service`: document, OCR, and transcription service on port `8001`
- `vectordb-service`: embedding, indexing, and retrieval service on port `8002`
- `core_app`: web UI on port `8190` and API on port `8180`

The containers mount local data from this repository so your projects, config, and cached models survive restarts.

## Prerequisites

Before starting the containers, make sure the following are in place:

1. Docker Desktop or Docker Engine with Compose support.
2. An `.env` file in the repository root. Start from `.env.example` and fill in the required API keys.
3. Ollama running on the host machine at `http://localhost:11434`.
4. The Ollama models referenced by `config.yaml` available on the host.

Recommended Ollama models from the current config:

- `qwen3-vl:4b`
- `qwen3:8b`
- `qwen3:14b`

Example host setup:

```bash
cp .env.example .env
ollama serve
ollama pull qwen3-vl:4b
ollama pull qwen3:8b
ollama pull qwen3:14b
```

## Start The Containers

From the repository root:

```bash
docker compose up -d
```

To verify that everything started correctly:

```bash
docker compose ps
docker compose logs -f core_app
```

If you want to rebuild or re-pull the images first:

```bash
docker compose pull
docker compose up -d
```

## Access Points

Once the stack is running:

- UI: `http://localhost:8190`
- Admin UI: `http://localhost:8190/admin`
- API: `http://localhost:8180`
- Qdrant: `http://localhost:6333`

User accounts, allowed models, and application behavior are configured in `config.yaml`.

## How To Use It

There are two common ways to work with the system.

### 1. Use The Web UI

Open `http://localhost:8190` and sign in with a user defined in `config.yaml`. The UI is the main entry point for chat and project-level interaction.

### 2. Use The API And Mounted Project Folders

Project data is mounted from the local `projects/` directory into the containers. A typical flow is:

1. Create a project.
2. Upload files or place files under that project's `input/` directory.
3. Trigger extraction or report generation through the API or UI.
4. Review generated outputs under `projects/<project>/processed/`.

Example API flow:

```bash
curl -X POST "http://localhost:8180/projects" \
  -H "Content-Type: application/json" \
  -d '{"project_name": "demo01"}'

curl -X POST "http://localhost:8180/projects/demo01/files" \
  -F "files=@/absolute/path/to/document.pdf"
```

If you prefer to manage files directly on disk, create a project folder under `projects/` and place source material into its `input/` subfolders. The existing `projects/eyerising/` directory shows the expected structure.

## Important Mounted Paths

- `./projects` -> application project workspace
- `./investment_data` -> shared investment datasets used by `core_app`
- `./config.yaml` -> runtime configuration
- `./.data/qdrant` -> persistent Qdrant storage
- `./.data/vectordb` -> persistent vector database artifacts
- `./.huggingface_cache`, `./.whisper_cache`, `./.media_cache` -> model and media caches

## Operational Commands

Start services:

```bash
docker compose up -d
```

View logs for all services:

```bash
docker compose logs -f
```

Restart a single service:

```bash
docker compose restart core_app
```

Stop the stack:

```bash
docker compose down
```

Stop the stack and remove persistent container network resources while keeping local volumes:

```bash
docker compose down --remove-orphans
```

## Notes

- This repository uses published images such as `auslei/investor_iq_core_app:latest` rather than building from source locally.
- The containers expect Ollama to be reachable from inside Docker via `host.docker.internal:11434`.
- If model downloads are slow on first run, allow time for the cache directories to populate.
- If you need to change login users, models, or default report behavior, update `config.yaml` and restart the affected services.
