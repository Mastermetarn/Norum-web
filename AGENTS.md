# Norum-web project context

## Purpose and repositories

- This repository contains the Norum portfolio landing page plus the production Caddy and Docker Compose configuration for `norum.se`.
- The rate-limit demo is a separate sibling repository at `../rate-limit-demo` with remote `https://github.com/Mastermetarn/rate-limit-demo.git`.
- Send is also a separate application and is consumed only as `ghcr.io/mastermetarn/send:latest` from this repository.
- Do not copy the rate-limit application source back into Norum-web. Keep the two repositories independently buildable and deployable.

## Production architecture

- `compose.yml` is the production stack and pulls these images:
  - `ghcr.io/mastermetarn/norum-web:latest` as `website`, port 80 internally.
  - `ghcr.io/mastermetarn/send:latest` as `send`, port 3000 internally.
  - `ghcr.io/mastermetarn/rate-limit-demo:latest` as `rate-limit-demo`, port 3000 internally.
  - `caddy:2` publishes host ports 80 and 443.
- Multiple services may use port 3000 because each container has its own network namespace. Do not publish their port 3000 on the VPS; Caddy reaches them by Compose service name.
- Production routing in `caddy/Caddyfile` is:
  - `/rate-limit` redirects to `/rate-limit/`.
  - `/rate-limit/*` uses `handle_path`, strips the prefix, and proxies to `rate-limit-demo:3000`.
  - `/send*` proxies to `send:3000` without stripping its prefix.
  - Everything else proxies to `website:80`.

## Local development

- Use both Compose files from the Norum-web root:

  ```sh
  docker compose -f compose.yml -f compose.local.yml up -d --build
  ```

- `compose.local.yml` builds the landing page from `./landingpage` and the rate-limit app from `../rate-limit-demo`.
- `caddy/Caddyfile.local` serves plain HTTP on `localhost`; do not use it in production.
- Local URLs are `http://localhost` and `http://localhost/rate-limit/`.
- Stop the local stack with:

  ```sh
  docker compose -f compose.yml -f compose.local.yml down
  ```

## Deployment workflow

- The VPS deployment directory is `/var/server` and is currently not a Git checkout. `git pull` there will fail. Copy updated `compose.yml` and `caddy/Caddyfile` to `/var/server` when those files change.
- Build and push Linux AMD64 images manually from each source directory:

  ```sh
  docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/rate-limit-demo:latest --push .
  docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/norum-web:latest --push .
  ```

  Run the second command from `landingpage/`.
- On the VPS, use only `compose.yml`:

  ```sh
  docker compose pull
  docker compose up -d --remove-orphans
  docker compose ps
  ```

- Pulling images does not reload an already-running Caddy process. After changing the mounted Caddyfile, run:

  ```sh
  docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
  docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
  ```

  A Caddy restart is an acceptable fallback.

## Rate-limit demo behavior

- The demo is an Express and vanilla JavaScript application.
- It uses an in-memory token bucket per automatically issued, hidden demo API key.
- Bucket capacity is 100 tokens and refill is continuous at 100 tokens/minute.
- The UI can send one request or generate 20 requests/second until stopped.
- Empty buckets return HTTP 429 and `Retry-After`; missing or unknown keys return HTTP 401.
- The frontend extrapolates refill at about 1.67 tokens/second between requests, then synchronizes to the authoritative server value after every response.
- Rate state is intentionally not persistent and resets when the container restarts.

## UI direction

- Keep the rate-limit demo deliberately simple and human-looking rather than highly polished or template-like.
- Prefer a white background, restrained typography, minimal borders, few decorative effects, and no unnecessary cards, gradients, pills, or large shadows.
- The available-token bar is the main visual element and should remain prominent.
- API keys and curl examples are implementation details and should not be shown in the main demo UI.
- Keep the portfolio link near the top of the demo.

## Repository hygiene

- Do not commit `caddy/data/`, `caddy/config/`, `.DS_Store`, dependency directories, or other runtime-generated files.
- Preserve unrelated user work such as `profile-app/` unless the task explicitly includes it.
- Validate Compose changes with `docker compose config` and validate Caddyfiles with `caddy validate` before deployment.
