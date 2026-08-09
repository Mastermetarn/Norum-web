# Norum-web

Norum.se portfolio and Caddy deployment configuration.

The production Compose stack exposes three applications:

- `/` - portfolio website
- `/send` - Send application
- `/rate-limit/` - interactive API rate limiting demo

## Build the portfolio image

Run from `landingpage/`:

```sh
docker build -t ghcr.io/mastermetarn/norum-web:latest .
```

Run locally:

```sh
docker run --rm -p 8080:80 ghcr.io/mastermetarn/norum-web:latest
```

Publish to GHCR:

```sh
docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/norum-web:latest --push .
```

## Rate limit demo

The standalone app source lives in the sibling repository `../rate-limit-demo`. Its own README covers the API, tests, local development, Docker build, and manual publication to `ghcr.io/mastermetarn/rate-limit-demo:latest`.

## Deploy on the VPS

```sh
docker compose pull
docker compose up -d
```

Caddy redirects `/rate-limit` to `/rate-limit/`, removes that prefix, and proxies the request to the standalone app on port 3000.

## Run the full stack locally

The local override builds the portfolio and rate limit demo from source and serves them through Caddy without HTTPS:

```sh
docker compose -f compose.yml -f compose.local.yml up -d --build
```

Open the portfolio at <http://localhost> and the demo at <http://localhost/rate-limit/>.

Stop the local stack with:

```sh
docker compose -f compose.yml -f compose.local.yml down
```
