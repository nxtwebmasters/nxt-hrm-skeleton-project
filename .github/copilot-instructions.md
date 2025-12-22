# Copilot instructions — nxt-hrm-skeleton-project

Purpose
- Help AI coding agents become productive quickly in this repository by describing architecture, workflows, conventions, and where to change behavior.

Big picture
- This repository is a skeleton that composes three runtime pieces via Docker Compose: a prebuilt frontend image (HRM dashboard), a prebuilt backend API image, and a MongoDB service. See [docker-compose.yml](docker-compose.yml#L1-L200).
- Backend expects an API prefix and environment configuration from the env file at [hrm-backend.env](hrm-backend.env#L1-L200). Key runtime integrations are `MONGO_URI`, `FRONTEND_URL`, `FILE_BASE_URL`, and `OPENAI_API_KEY`.

Developer workflows (concrete)
- Start full stack (Docker Desktop / WSL2 recommended on Windows):

  ```powershell
  docker compose --env-file hrm-backend.env up -d
  docker compose logs -f hrm-mongo-apis
  ```

- If you need to develop the backend locally instead of using the published image, replace the `image:` entry with a `build:` section in an override compose file or `docker compose -f docker-compose.yml -f docker-compose.override.yml up --build`. The backend image in this repo is `pandanxt/nxt-hrm-backend:...` so local work requires a local backend repo or Dockerfile.

Important conventions and patterns (project-specific)
- Service names and hostnames are used in `MONGO_URI` and healthchecks (e.g. `mongodb` container). The compose file relies on container hostnames rather than external hostnames — use `mongodb` when containers resolve each other. See the Mongo URI in [hrm-backend.env](hrm-backend.env#L1-L200).
- Uploads are persisted to a Docker volume `uploads_data` and mounted at `/app/uploads` inside the backend container. Use this path when scripting file maintenance or inspecting uploaded assets.
- Authentication/session separation: `JWT_SECRET` is used for tokens; `EXPRESS_SESSION_SECRET` is a separate, long secret used for express-session (WebAuthn challenges). Keep these distinct.
- `LOGIN_IDENTIFIERS` is a comma-separated list of fields that the backend accepts for login (e.g., `email,empCode,work_phone`) — update only via environment or seed logic.

Healthchecks and ordering
- The backend service uses `depends_on` with a healthcheck on `mongodb`. The backend expects a `/api/health` endpoint for its own health probe (see [docker-compose.yml](docker-compose.yml#L1-L200)).

Secrets and safety
- `hrm-backend.env` contains many example secrets and comments about production practices; do NOT commit production secrets or OpenAI keys to public repos. See [hrm-backend.env](hrm-backend.env#L1-L200) for the list of environment keys the backend reads.

Quick troubleshooting tips
- If backend fails to connect to DB, verify `MONGO_URI` hosts and credentials, and that `mongodb` is healthy: `docker compose ps` and `docker compose logs mongodb`.
- To inspect the uploads volume:

  ```bash
  docker compose exec hrm-mongo-apis ls -la /app/uploads
  docker volume inspect nxt-hrm-skeleton-project_uploads_data
  ```

Where to edit code
- This skeleton references published images; if you need to change backend or frontend source you must either: (a) clone the respective source repo and run it locally, or (b) add a `build:` section in compose so Docker builds a local image.

Key files to review
- [docker-compose.yml](docker-compose.yml#L1-L200) — service wiring, volumes, healthchecks
- [hrm-backend.env](hrm-backend.env#L1-L200) — runtime configuration and secrets
- [data/scripts/init.sql](data/scripts/init.sql#L1-L50) — DB init script (minimal here)

If anything above is unclear or you want me to: (1) add a `docker-compose.override.yml` example for local backend development, or (2) redact/rotate secrets and add a `.env.example`, tell me which and I'll add it.
