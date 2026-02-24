## MEAN CRUD Tutorials App

Full-stack CRUD application built with **MongoDB + Express + Angular 15 + Node.js**.  
It manages a collection of **tutorials**:

- `title` (string)
- `description` (string)
- `published` (boolean)

Users can **create / list / update / delete** tutorials and **search by title**.

---

## Project structure

```
.
├─ backend/      # Node.js + Express REST API
├─ frontend/     # Angular 15 UI
├─ NGINX/        # Reverse proxy config used by docker-compose
└─ docker-compose.yml
```

## Ports & URLs

- **Backend API**: `http://localhost:8081`
- **Frontend (Angular dev server)**: `http://localhost:4200`
- **Nginx reverse proxy (Docker)**: `http://localhost`  
  - UI: `/`
  - API: `/api/*` → backend

## API endpoints

Base path: `/api/tutorials`

- `GET /api/tutorials` (supports query: `?title=...`)
- `GET /api/tutorials/:id`
- `POST /api/tutorials`
- `PUT /api/tutorials/:id`
- `DELETE /api/tutorials/:id`
- `DELETE /api/tutorials`
- `GET /api/tutorials/published`

---

## Prerequisites

Choose one of the following run modes:

### Option A (recommended): Docker

- Docker Desktop (Docker Compose v2)

### Option B: Local dev (no Docker)

- Node.js **18+** and npm
- MongoDB running locally (or in Docker)
- Angular CLI (optional; `npm run start` will work without global install)

---

## Setup & Run (Option A: Docker Compose)

This repo includes `docker-compose.yml` which starts:
- `mongo` (database)
- `backend` (Express API)
- `frontend` (Angular dev server)
- `nginx` (reverse proxy: routes `/api` to backend and `/` to frontend)

### Step 1: Start the stack

From repo root:

```bash
docker compose up -d
```

### Step 2: Open the app

- App (via nginx): `http://localhost`
- API (via nginx): `http://localhost/api/tutorials`

### Step 3: Stop the stack

```bash
docker compose down
```

### Notes (Docker)

- **MongoDB data persists** in the named volume `mongo_data`.
- `backend/.env` is used by Compose to set `MONGO_URI` inside the backend container.
- The Compose file is configured to use published images (`alaypatel212/backend`, `alaypatel212/frontend`). If you want to build locally, edit `docker-compose.yml` to enable the `build:` lines for `backend` and `frontend` and remove/replace the `image:` lines, then run:

```bash
docker compose up -d --build
```

---

## Setup & Run (Option B: Local development)

### Step 1: Start MongoDB

If you already have MongoDB installed locally, ensure it’s running on `mongodb://localhost:27017`.

Or run MongoDB in Docker only:

```bash
docker run --name dd-mongo -d -p 27017:27017 mongo
```

### Step 2: Run the backend (Express API)

The backend reads Mongo connection via environment variable **`MONGO_URI`** (see `backend/app/config/db.config.js`).

In one terminal:

```bash
cd backend
npm install
```

Set environment variables, then start the server:

**Windows PowerShell**

```powershell
$env:MONGO_URI="mongodb://localhost:27017/dd_db"
$env:PORT="8081"
node server.js
```

**macOS / Linux**

```bash
export MONGO_URI="mongodb://localhost:27017/dd_db"
export PORT="8081"
node server.js
```

Verify:
- `http://localhost:8081/` → should return a welcome JSON message

### Step 3: Run the frontend (Angular)

In a second terminal:

```bash
cd frontend
npm install
npm start
```

Open:
- `http://localhost:4200`

### Important: `/api` routing in local dev

The Angular app calls the API using a relative URL: `/api/tutorials` (see `frontend/src/app/services/tutorial.service.ts`).  
That works automatically when you access the app through nginx (Docker option), but **it needs a proxy** when running `ng serve` directly.

Create `frontend/proxy.conf.json` with:

```json
{
  "/api": {
    "target": "http://localhost:8081",
    "secure": false,
    "changeOrigin": true
  }
}
```

Then start Angular using:

```bash
cd frontend
npm start -- --proxy-config proxy.conf.json
```

---

## Deployment (production)

You have a few common choices. Pick the one that matches your environment.

### Option 1: Deploy with Docker (single server/VM)

1. Install Docker + Docker Compose on the server.
2. Copy this repo to the server (or clone it).
3. Ensure backend has the right MongoDB connection:
   - If using the Compose Mongo container, `backend/.env` can stay as:
     - `MONGO_URI=mongodb://mongo:27017/dd_db`
   - If using an external MongoDB (Atlas), set:
     - `MONGO_URI=<your Mongo connection string>`
4. Start:

```bash
docker compose up -d
```

Expose/allow inbound traffic to port **80** on the server and open:
- `http://<server-ip-or-domain>/`

### Option 2: Deploy without Docker (build frontend + run API)

**Backend**

1. On the server, set environment variables:
   - `MONGO_URI` (required)
   - `PORT` (optional; defaults to `8081`)
2. Install and run backend:

```bash
cd backend
npm ci
node server.js
```

For long-running production processes, use a process manager (e.g. `pm2`) or systemd.

**Frontend**

1. Build Angular:

```bash
cd frontend
npm ci
npm run build
```

2. Serve the built files from `frontend/dist/angular-15-crud/` using a web server (Nginx/Apache).
3. Configure your web server to proxy `/api/*` to your backend (e.g. `http://127.0.0.1:8081`).

---

## Troubleshooting

- **Backend cannot connect to MongoDB**
  - Confirm `MONGO_URI` is set and reachable.
  - Local Mongo: `mongodb://localhost:27017/dd_db`
  - Docker Compose Mongo from backend container: `mongodb://mongo:27017/dd_db`
- **Frontend shows blank list / API calls fail**
  - If running via Docker+nginx: open `http://localhost` (not `http://localhost:4200`).
  - If running `ng serve` directly: ensure you started with the proxy config as described above.
