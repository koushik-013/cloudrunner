# CloudRunner

CloudRunner is a full-stack web platform that lets university teachers manage NixOS configuration files for their students. It pairs a Rust/Axum REST API with a SvelteKit + Deno frontend, backed by PostgreSQL and fronted by Caddy for TLS and reverse proxying.

## Architecture

```
                ┌──────────────┐
   Browser ───▶ │    Caddy     │  (TLS termination, reverse proxy)
                └──────┬───────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
  ┌───────────────┐        ┌────────────────┐
  │ cloud-frontend │        │  cloud-backend  │
  │ SvelteKit/Deno │──API──▶│  Axum (Rust)    │
  └───────────────┘        └────────┬────────┘
                                     ▼
                              ┌────────────┐
                              │ PostgreSQL │
                              └────────────┘
```

- **`cloud-backend/`** — Axum REST API. Handles teacher authentication (JWT) and CRUD for `.nix` configuration files. See [`cloud-backend/README.md`](cloud-backend/README.md) for API reference and setup.
- **`cloud-frontend/`** — SvelteKit app running on the Deno runtime, styled with Tailwind CSS. See [`cloud-frontend/README.md`](cloud-frontend/README.md) for pages and dev workflow.
- **`compose.yml`** — Docker Compose stack wiring together `backend`, `frontend`, `db` (Postgres), and `caddy`.
- **`Caddyfile`** — Reverse proxy rules: `/api/*` and `/health` go to the backend, everything else to the frontend SPA.

## Features

- **Authentication** — registration, login/logout, JWT-based sessions, password reset flow
- **Configuration management** — upload, list, view, edit, and delete NixOS `.nix` config files per teacher
- **Containerized deployment** — one-command startup for the full stack via Docker Compose, with automatic HTTPS via Caddy

## Tech Stack

| Layer      | Technology |
|------------|------------|
| Backend    | Rust, Axum, SQLx, PostgreSQL, JWT (`jsonwebtoken`), `bcrypt` |
| Frontend   | SvelteKit 5, Deno, TypeScript, Tailwind CSS 4 |
| Proxy/TLS  | Caddy |
| Database   | PostgreSQL |
| Deployment | Docker Compose |

## Quick Start (Docker Compose)

This is the fastest way to run the whole stack:

```bash
git clone <repo-url>
cd cloudrunner-main
docker compose up --build
```

This starts:
- `db` — PostgreSQL, with data persisted in the `pg_data` volume
- `backend` — Axum API (runs migrations automatically on startup)
- `frontend` — SvelteKit app
- `caddy` — reverse proxy, exposed on ports `80` and `443`

Once running, the app is available through Caddy at `http://localhost`.

> **Note:** the default `compose.yml` uses a hardcoded Postgres password (`supersecretpassword`). Change this before deploying anywhere beyond local development.

## Local Development (without Docker)

Run backend and frontend separately for hot-reloading during development.

### Backend

```bash
cd cloud-backend
cp .env.example .env   # configure DATABASE_URL, JWT_SECRET, etc.
psql -U postgres -c "CREATE DATABASE university_nixos;"
psql -U postgres -d university_nixos -f schema.sql
cargo run
```

### Frontend

```bash
cd cloud-frontend
cp .env.example .env   # point PUBLIC_API_URL / VITE_API_URL at the backend
deno task dev
```

The frontend dev server defaults to `http://localhost:5173` and expects the backend at `http://localhost:8080` unless configured otherwise.

Full setup details, environment variables, and API endpoint documentation are in each sub-project's README:
- [`cloud-backend/README.md`](cloud-backend/README.md)
- [`cloud-frontend/README.md`](cloud-frontend/README.md)

## License

MIT
