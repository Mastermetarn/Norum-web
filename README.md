# Norum-web
Norum.se website


Build 
docker build -t ghcr.io/mastermetarn/norum-web:latest .

Run local

docker run --rm -p 8080:80 ghcr.io/mastermetarn/norum-web:latest

Upload container to ghcr:

docker buildx build --platform linux/amd64 -t ghcr.io/mastermetarn/norum-web:latest --push .

Run on VPS:

docker compose pull
docker compose up -d --build