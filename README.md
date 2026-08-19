# Norum-web

The source and deployment configuration for [norum.se](https://norum.se), a personal software portfolio hosting several web applications behind a single HTTPS endpoint.

## Live applications

| Application           | Description                                    | Link                                                 |
| --------------------- | ---------------------------------------------- | ---------------------------------------------------- |
| Portfolio             | Personal portfolio and project overview        | [norum.se](https://norum.se)                         |
| Send                  | End-to-end encrypted message sharing           | [norum.se/send](https://norum.se/send)               |
| Rate Limit Demo       | Interactive demonstration of API rate limiting | [norum.se/rate-limit/](https://norum.se/rate-limit/) |
| Multiplayer Blackjack | Real-time multiplayer KTH project              | [norum.se/blackjack/](https://norum.se/blackjack/)   |

## About the project

Norum-web contains the portfolio landing page and the infrastructure used to combine four applications into one deployment.

The repository includes:

- A static portfolio served by Nginx
- A Docker Compose production stack
- Caddy reverse-proxy and HTTPS configuration
- A local Compose override for running the complete stack
- Routing for regular HTTP and WebSocket traffic

Send, the rate-limit demo, and Blackjack are maintained as separate application codebases. Their production images are published to GitHub Container Registry and consumed by this deployment.

## Architecture

```text
Browser
   |
   | HTTPS
   v
Caddy
   |-- /             -> Portfolio (Nginx)
   |-- /send         -> Send
   |-- /rate-limit/  -> Rate Limit Demo
   `-- /blackjack/   -> Multiplayer Blackjack
```

Caddy terminates TLS and routes requests to the appropriate service over the private Docker Compose network. The application containers do not publish their internal ports directly on the production host.

Production images are built for Linux AMD64, stored in GitHub Container Registry, and pulled by the production Compose stack.

## Technology

- Docker and Docker Compose
- Caddy
- Nginx
- GitHub Container Registry
- Linux
- HTTPS/TLS
- Reverse-proxy routing
- WebSocket proxying

## Applications

### Portfolio

The portfolio is a static website served by Nginx. It presents selected software projects and provides links to their live deployments.

### Send

Send is an end-to-end encrypted message-sharing application. Caddy forwards requests under `/send` to its standalone application container without removing the path prefix.

### Rate Limit Demo

The rate-limit demo is an interactive Express application demonstrating a token-bucket rate limiter.

It supports:

- Automatically issued demo API keys
- A bucket capacity of 100 tokens
- Continuous refill at 100 tokens per minute
- Single-request and request-flood controls
- HTTP `429 Too Many Requests` responses
- `Retry-After` headers
- Live request and token statistics

Caddy removes the `/rate-limit/` prefix before forwarding requests to the application.

### Multiplayer Blackjack

Blackjack is a real-time multiplayer KTH project with accounts, persistent balances, lobbies, and synchronized game state.

The application uses Socket.IO for real-time communication. Caddy removes the `/blackjack/` prefix and proxies both HTTP and WebSocket traffic to the application container.

Demo data is cleared when the production container restarts and automatically each night.

## Repository structure

```text
.
|-- landingpage/
|   |-- html/              # Portfolio pages and static assets
|   |-- nginx.conf         # Nginx configuration
|   `-- Dockerfile         # Portfolio container image
|-- caddy/
|   |-- Caddyfile          # Production HTTPS and routing configuration
|   `-- Caddyfile.local    # Local HTTP configuration
|-- compose.yml            # Production stack
|-- compose.local.yml      # Local build and routing overrides
|-- DEPLOYMENT.md          # Publishing and deployment runbook
`-- README.md
```

## Run the portfolio locally

Build the portfolio image from the repository root:

```sh
docker build -t norum-web:local ./landingpage
```

Start the container:

```sh
docker run --rm -p 8080:80 norum-web:local
```

Open [http://localhost:8080](http://localhost:8080).

## Run the complete stack locally

The local environment expects the Send, rate-limit demo, and Blackjack repositories at the sibling paths configured in `compose.local.yml`.

Start the complete stack:

```sh
docker compose -f compose.yml -f compose.local.yml up -d --build
```

The applications will be available at:

- Portfolio: [http://localhost](http://localhost)
- Send: [http://localhost/send](http://localhost/send)
- Rate Limit Demo: [http://localhost/rate-limit/](http://localhost/rate-limit/)
- Blackjack: [http://localhost/blackjack/](http://localhost/blackjack/)

Stop the stack:

```sh
docker compose -f compose.yml -f compose.local.yml down
```

## Deployment

Image publishing, production deployment, registry authentication, and operational instructions are documented separately in [DEPLOYMENT.md](DEPLOYMENT.md).
