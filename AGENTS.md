# AGENTS.md

## Project Shape
- This repo is a fork/custom deployment of upstream `https://github.com/harry0703/MoneyPrinterTurbo`; deploys use `origin` at `https://github.com/felipebarbosavasconcelos13-coder/MoneyPrinterTurbo.git`.
- Main WebUI entrypoint is `webui/Main.py` via Streamlit on port `8501`.
- Main API entrypoint is `main.py`, which runs `app.asgi:app` with `config.listen_host` and `config.listen_port`; defaults are `0.0.0.0:8080` from `app/config/config.py`.
- API routes included in production are wired in `app/router.py`; `app/controllers/ping.py` defines `/ping` but is not included, so `/ping` returning `404` is expected unless router wiring changes.
- V1 API routes use prefix `/api/v1` from `app/controllers/v1/base.py`; API docs are available at `/docs` and `/openapi.json`.

## Commands
- Install/sync preferred environment with `uv sync --frozen` using Python `>=3.11,<3.13`; `requirements.txt` exists for Docker/legacy pip.
- Run WebUI locally on Windows with `./webui.bat`; on Unix-like systems use `uv run streamlit run ./webui/Main.py --browser.gatherUsageStats=False`.
- Run API locally with `uv run python main.py`.
- Run all tests with `python -m unittest discover -s test`.
- Run one test file with `python -m unittest test/services/test_video.py`; run one method with `python -m unittest test.services.test_video.TestVideoService.test_preprocess_video`.
- Validate Compose before deploy edits with `docker compose -f docker-compose.yml config`.

## Coolify Deployment
- The current working Coolify deployment depends on explicit Traefik labels in `docker-compose.yml`; do not remove them or switch back to automatic `coolify.managed=true` routing without retesting.
- The fix for `503 no available server` was commit `ab939f8`, adding explicit routers/services and attaching both services to the external Docker network `coolify`.
- WebUI domain routes to service `webui` on container port `8501`: `https://moneyprinterturbo.genialsolucoesdigitais.com.br/`.
- API domain routes to service `api` on container port `8080`: `https://api-moneyprinterturbo.genialsolucoesdigitais.com.br/`.
- Post-deploy smoke checks: WebUI `/` should return `HTTP 200`; API `/docs` and `/openapi.json` should return `HTTP 200`; `/ping` may return `404` because it is not registered.
- The upstream/local Compose style using `ports: "127.0.0.1:8501:8501"` is for local Docker and is not suitable for Coolify/Traefik in this repo.
- Single Dockerfile mode only starts the Streamlit WebUI via Dockerfile `CMD`; use Docker Compose mode when both WebUI and API must be deployed.

## Config And Secrets
- `.dockerignore` excludes `config.toml`, so Docker/Coolify images create `config.toml` from `config.example.toml` at runtime if no mounted file exists.
- `config.toml` is gitignored and may contain real API keys locally; never stage it or copy secrets into docs.
- `.env` and `.env.*` are ignored by Docker context; do not rely on them being copied into images.
- Current untracked `.agents/` and `skills-lock.json` are local OpenCode skill artifacts; do not commit them unless explicitly requested.

## Verification Notes
- A successful Coolify deployment can still show app status `running:unknown` because health checks are disabled; verify with HTTP instead of status text alone.
- If Traefik returns `503 no available server`, first inspect Compose labels/networking and Coolify deployment logs; container build/start may still be successful.
