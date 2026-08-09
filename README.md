# Norum-web

Norum.se portfolio and Caddy deployment configuration.

The production Compose stack exposes four applications:

- `/` - portfolio website
- `/send` - Send application
- `/rate-limit/` - interactive API rate limiting demo
- `/blackjack/` - real-time multiplayer blackjack school project

## Publish application images to GHCR

The VPS runs Linux AMD64 images from GitHub Container Registry. Caddy uses the
official `caddy:2` image and is not built or published from this repository.

Create a GitHub personal access token (classic) with the `write:packages` scope,
store it temporarily in `GHCR_TOKEN`, and sign in:

```sh
export GHCR_TOKEN=YOUR_GITHUB_TOKEN
printf '%s' "$GHCR_TOKEN" | docker login ghcr.io -u Mastermetarn --password-stdin
```

From the Norum-web root, build and push every application image used by
`compose.yml`:

```sh
docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/norum-web:latest --push ./landingpage
docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/send:latest --push ../Send/application
docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/rate-limit-demo:latest --push ../rate-limit-demo
docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/adrino-alesive-project:latest --push ../Blackjack
```

To publish only Send after changing it:

```sh
docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/send:latest --push ../Send/application
```

Verify the published AMD64 manifests:

```sh
docker buildx imagetools inspect ghcr.io/mastermetarn/norum-web:latest
docker buildx imagetools inspect ghcr.io/mastermetarn/send:latest
docker buildx imagetools inspect ghcr.io/mastermetarn/rate-limit-demo:latest
docker buildx imagetools inspect ghcr.io/mastermetarn/adrino-alesive-project:latest
```

The first manually published GHCR package is private by default. Configure its
visibility and repository access in GitHub Packages as needed. Do not commit or
paste the token into this repository; unset it when publishing is complete:

```sh
unset GHCR_TOKEN
```

## Build the portfolio image locally

Run from `landingpage/`:

```sh
docker build -t ghcr.io/mastermetarn/norum-web:latest .
```

Run locally:

```sh
docker run --rm -p 8080:80 ghcr.io/mastermetarn/norum-web:latest
```

## Rate limit demo

The standalone app source lives in the sibling repository `../rate-limit-demo`.
Its own README covers the API, tests, and local development.

## Send

The standalone Send source lives in `../Send/application`. Its production image
is `ghcr.io/mastermetarn/send:latest`, and Caddy proxies `/send*` without
stripping the prefix.

## Blackjack school project

The standalone app source lives in the sibling directory `../Blackjack` and is
published as `ghcr.io/mastermetarn/adrino-alesive-project:latest`.

The app listens over plain HTTP inside the Compose network. Caddy terminates HTTPS, removes the `/blackjack` prefix, and proxies HTTP and Socket.IO traffic to the app on port 8989.

The Compose service sets `RESET_DATA_ON_START=true`. Starting or restarting the
Blackjack container clears its account, game, and session databases before the
server starts. The application also clears all data nightly at 03:00
Europe/Stockholm time.

## Deploy on the VPS

```sh
docker compose pull
docker compose up -d
```

Caddy redirects prefix-less app URLs to their trailing-slash forms, removes the prefixes, and proxies each request to its standalone app.

## Run the full stack locally

The local override builds the portfolio, Send, rate limit demo, and Blackjack
from their local source directories and serves them through Caddy without HTTPS:

```sh
docker compose -f compose.yml -f compose.local.yml up -d --build
```

Open the portfolio at <http://localhost>, Send at <http://localhost/send>, the
rate limit demo at <http://localhost/rate-limit/>, and blackjack at
<http://localhost/blackjack/>.

Stop the local stack with:

```sh
docker compose -f compose.yml -f compose.local.yml down
```
