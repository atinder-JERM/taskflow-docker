# TaskFlow — Docker & Docker Compose Hands-on Project

A small full-stack task manager used to teach Dockerfiles, image building, Docker Compose,
PostgreSQL persistence, Redis caching, Docker Hub publishing, and deployment from pulled images.

## Stack
- Frontend: React + Vite, served by Nginx
- Backend: Node.js + Express
- Database: PostgreSQL
- Cache: Redis
- Tests: Jest + Supertest

## Prerequisites
Docker Desktop (or Docker Engine + Compose plugin), Node.js 20+ (only needed to run tests locally),
and a Docker Hub account.

## 1. Run locally with Docker Compose (build from source)
Copy `.env.example` to `.env`, then:

```bash
docker compose up --build -d
docker compose ps
```

Open:
- Frontend: http://localhost:8080
- Backend health: http://localhost:3000/health

Stop:
```bash
docker compose down
```

PostgreSQL data is stored in a named volume. To remove the database data too:
```bash
docker compose down -v
```

## 2. Publish application images to Docker Hub
Replace `YOUR_DOCKERHUB_USERNAME` with your Docker Hub username.

```bash
docker login
docker build -t YOUR_DOCKERHUB_USERNAME/taskflow-backend:v1 ./backend
docker build -t YOUR_DOCKERHUB_USERNAME/taskflow-frontend:v1 ./frontend
docker push YOUR_DOCKERHUB_USERNAME/taskflow-backend:v1
docker push YOUR_DOCKERHUB_USERNAME/taskflow-frontend:v1
```

## 3. Deploy on another machine using only Docker Hub images
On the deployment machine, copy this repository's `docker-compose.deploy.yml` and `.env`
(or create them manually). Edit the two image names in `docker-compose.deploy.yml`.

```bash
docker compose -f docker-compose.deploy.yml pull
docker compose -f docker-compose.deploy.yml up -d
```

Open http://DEPLOYMENT_MACHINE_IP:8080

This deployment Compose file uses `image:` for the frontend and backend; it does not build
application images from source. PostgreSQL and Redis use official upstream images.

## 4. Run automated API tests
With the application already running:

```bash
cd tests
npm install
npm test
```

The test suite uses `BASE_URL` (default `http://localhost:3000`). It creates a unique user,
then checks registration, login, task CRUD, health, and Redis cache behaviour.

```bash
# Windows PowerShell example
$env:BASE_URL="http://localhost:3000"
npm test
```

## API overview
- `GET /health` — API, PostgreSQL and Redis status
- `POST /api/auth/register` — `{ "name": "...", "email": "...", "password": "..." }`
- `POST /api/auth/login` — `{ "email": "...", "password": "..." }`
- `GET /api/tasks` — authenticated; returns tasks (cached in Redis)
- `POST /api/tasks` — authenticated; `{ "title": "...", "description": "..." }`
- `PATCH /api/tasks/:id` — authenticated; accepts `title`, `description`, `status`
- `DELETE /api/tasks/:id` — authenticated

Send the login token as `Authorization: Bearer TOKEN`.

## Teaching checkpoints
1. Build and run frontend/backend separately.
2. Explain image layers, ports, logs, and container lifecycle.
3. Bring up all services with Compose.
4. Stop/recreate PostgreSQL and verify persisted data.
5. Push frontend/backend images to Docker Hub.
6. Delete local copies of those images, then deploy using the image-only Compose file.
7. Run the test suite and discuss failures.

## Important notes
- This is a teaching/demo project, not production-hardened software.
- Change all `.env` secrets before sharing or deploying.
- Do not commit `.env`; it is ignored by Git.
- For a real production deployment, use TLS, managed secrets, stronger auth controls,
  backups, migrations, non-root containers, and pinned image digests.
