# Mean Chat Docker Deployment Guide

This document explains how to run `mean-chat` with Docker, publish images to Docker Hub, and run the app on another machine.

## 1. What is Dockerized

The stack has 3 services:

- `frontend`: Angular + Ionic static build served by Nginx on port `80`
- `backend`: Node/Express API + Socket.IO on port `3000`
- `mongo`: MongoDB (container-internal, persisted with Docker volume)

`docker-compose.yml` connects them through the `chat-net` network.

## 2. Prerequisites

On any machine where you want to run it:

- Docker Desktop installed and running
- Linux containers mode enabled
- WSL2 backend enabled (on Windows)

## 3. First Run (Current Setup)

From project root (`mean-chat`):

```powershell
docker compose up --build -d
```

Check status:

```powershell
docker compose ps -a
```

Open:

- Frontend: `http://localhost`
- Backend API: `http://localhost:3000`

## 4. Environment Variables

Root `.env` in `mean-chat` is intentionally versioned in git for this project, so a new machine gets the same values after clone/pull.

Current variables:

```env
JWT_SECRET=your_long_random_secret
JWT_REFRESH_SECRET=your_long_random_refresh_secret
GEOAPIFY_API_KEY=your_geoapify_key
EMOJI_API_KEY=your_emoji_api_key
```

Notes:

- `JWT_*` variables are consumed by backend runtime.
- `GEOAPIFY_API_KEY` and `EMOJI_API_KEY` are injected into frontend during Docker build.
- `.env` is tracked by git in this repository (intentional for this setup).
- If you rotate any key/secret, update `.env`, commit, and pull on other machines.

## 5. Rebuild After Changes

If code or env changes:

```powershell
docker compose down
docker compose up --build -d
```

If you need a clean database reset:

```powershell
docker compose down -v
docker compose up --build -d
```

## 6. Swagger in Containers

Swagger is configured as **dev-only**.

- In production container mode (`NODE_ENV=production`), `/api-docs` is not exposed (expected `404`).
- If you need Swagger in container runtime, set backend `NODE_ENV` to non-production and include swagger deps as runtime deps.

## 7. Run on Another PC (Build from Source)

Recommended workflow:

1. Clone this repository on the target machine.

```powershell
git clone --recurse-submodules https://github.com/RealYonoveleta/mean-chat
cd mean-chat
```

2. If you already cloned without submodules, initialize them:

```powershell
git submodule update --init --recursive
```

Notes:

- If this repo has no submodules, these commands still work safely.
- Using `--recurse-submodules` avoids missing nested dependencies in repos that include submodules.

3. Install/start Docker Desktop.
4. Pull latest changes (including `.env` if updated):

```powershell
git pull
```

5. Run:

```powershell
docker compose up --build -d
```

This is usually the simplest and most reliable for development.

### 7.1 About `.env` on New PCs

Because `.env` is tracked in this repo, you do not need to rewrite variables manually on each machine.

Typical update flow:

```powershell
git pull
docker compose up --build -d
```

If secrets are changed in git, each machine only needs a new `git pull`.

## 8. Run on Another PC (Docker Hub Images)

If you want prebuilt images:

### 8.1 Push images from source machine

```powershell
docker login
docker tag mean-chat-backend <dockerhub-user>/mean-chat-backend:v1
docker tag mean-chat-frontend <dockerhub-user>/mean-chat-frontend:v1
docker push <dockerhub-user>/mean-chat-backend:v1
docker push <dockerhub-user>/mean-chat-frontend:v1
```

### 8.2 Use images on target machine

Adjust `docker-compose.yml` to use `image:` instead of `build:` for `backend` and `frontend`, then:

```powershell
docker compose up -d
```

Mongo can still use `mongo:4.4` image directly from Docker Hub.

## 9. Data Persistence and Migration

Mongo data is persisted in Docker volume `mongo_data`.

- Keeping `docker compose down` preserves data.
- `docker compose down -v` removes database data.

To move data to another machine, use `mongodump`/`mongorestore`.

Example backup (from host):

```powershell
docker exec mean-chat-mongo-1 mongodump --db mean-chat --out /tmp/dump
docker cp mean-chat-mongo-1:/tmp/dump ./dump
```

Example restore (on target):

```powershell
docker cp ./dump mean-chat-mongo-1:/tmp/dump
docker exec mean-chat-mongo-1 mongorestore --db mean-chat /tmp/dump/mean-chat
```

## 10. Troubleshooting

### Build appears stuck at "Building"

- Frontend `npm ci` may take ~1 minute with little output. Wait.
- Check daemon health:

```powershell
docker version
docker buildx ls
```

- If daemon is unresponsive:
  - Quit Docker Desktop
  - Run `wsl --shutdown`
  - Start Docker Desktop again

### Frontend loads but login form missing

- Usually stale browser cache or Ionic web components not hydrated in an old build.
- Hard refresh (`Ctrl+F5`) or open in incognito.
- Rebuild frontend image:

```powershell
docker compose up --build -d frontend
```

### Mongo exits with code 132

- CPU compatibility issue with newer Mongo images on some machines.
- Current compose uses `mongo:4.4` for compatibility.

## 11. Useful Commands

```powershell
# Start (detached)
docker compose up -d

# Rebuild all
docker compose up --build -d

# Stop services
docker compose down

# Stop and remove volumes
docker compose down -v

# Follow logs
docker compose logs -f

# Check status
docker compose ps -a
```

---

If you later want production-grade deployment (TLS, custom domain, reverse proxy, CI/CD), create a separate compose profile or deployment manifest instead of reusing local-dev settings directly.
