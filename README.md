# Mean Chat

Monorepo wrapper for the Mean Chat application.

This repository orchestrates two subprojects:

- mean-chat-back: Node.js, Express, Socket.IO, MongoDB
- mean-chat-front: Angular, Ionic

It also includes Docker Compose for a full local stack (frontend + backend + MongoDB).

## Repository Layout

- mean-chat-back: backend submodule
- mean-chat-front: frontend submodule
- docker-compose.yml: full stack orchestration
- .env: shared Docker variables (JWT + API keys)
- DOCKER_DEPLOYMENT_GUIDE.md: extended deployment and troubleshooting guide

## Clone

Because frontend and backend are Git submodules, clone with submodules enabled:

```bash
git clone --recurse-submodules https://github.com/RealYonoveleta/mean-chat
cd mean-chat
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Prerequisites

- Node.js 20+ (for local non-Docker development)
- npm
- Docker Desktop (for containerized workflow)

## Quick Start (Docker)

1. Ensure root .env has required values:

```env
JWT_SECRET=<your-secret>
JWT_REFRESH_SECRET=<your-refresh-secret>
GEOAPIFY_API_KEY=<your-geoapify-key>
EMOJI_API_KEY=<your-emoji-api-key>
```

2. Start the stack:

```bash
docker compose up --build -d
```

3. Open:

- Frontend: http://localhost
- Backend: http://localhost:3000

Useful commands:

```bash
docker compose ps -a
docker compose logs -f
docker compose down
docker compose down -v
```

## Quick Start (Classic Local Development)

You can run frontend and backend without Docker.

From repository root:

```bash
npm run dev
```

Available root scripts:

- npm run dev: starts backend and frontend in separate terminals (Windows-oriented)
- npm run dev:back: backend only
- npm run dev:front: frontend only

Important:

- Backend local run uses mean-chat-back/.env.
- If Docker backend is already running on port 3000, stop it before local backend start.

## Environment Notes

- Docker backend gets MONGO_URI from docker-compose.yml (mongodb://mongo:27017/mean-chat).
- JWT auth uses two secrets: JWT_SECRET (access token) and JWT_REFRESH_SECRET (refresh token).
- Frontend Docker build injects GEOAPIFY_API_KEY and EMOJI_API_KEY into production environment.ts.
- Angular local dev uses environment.development.ts.

## Authentication Notes

- Access token expires in 1 hour.
- Refresh token expires in 7 days and is rotated via /auth/refresh.
- Frontend retries failed 401 requests once after refreshing the token.

## Current Architecture Notes

- Swagger endpoint is dev-only in backend (disabled when NODE_ENV=production).
- Mongo image in compose is pinned to mongo:4.4 for broader compatibility.
- Frontend theme uses Ionic dark.system plus project color tokens.

## Additional Documentation

- Full deployment guide: DOCKER_DEPLOYMENT_GUIDE.md
- Backend-specific docs: mean-chat-back/README.md
- Frontend-specific docs: mean-chat-front/README.md
