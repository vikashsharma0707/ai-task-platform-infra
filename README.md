# AI Task Processing Platform

Production-ready web application where authenticated users create AI-style text-processing tasks, execute them asynchronously via Redis queue + Python worker, and monitor status/logs/results.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + Vite + TypeScript + Tailwind CSS |
| Backend API | Node.js + Express.js (ES Modules) |
| Worker | Python 3.11 + RQ/Redis polling |
| Database | MongoDB + Mongoose |
| Queue | Redis — BullMQ (producer) + custom poller (consumer) |
| Auth | JWT + bcrypt |
| Containers | Docker multi-stage builds, non-root user |
| Orchestration | Kubernetes + Kustomize (base + overlays) |
| GitOps | Argo CD (auto-sync) |
| CI/CD | GitHub Actions |

## Local Development

### Prerequisites
- Node.js 20+, Python 3.11+, Docker, Docker Compose

### Quickstart (Docker Compose)
```bash
cp .env.example .env
# Edit .env — set JWT_SECRET at minimum

docker compose up --build
```

Services:
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000
- MongoDB: localhost:27017
- Redis: localhost:6379

### Run Without Docker

**Backend:**
```bash
cd backend
cp .env.example .env
npm install
npm run dev
```

**Worker:**
```bash
cd worker
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python -m src.worker
```

**Frontend:**
```bash
npm install
npm run dev
# Vite dev server at http://localhost:5173
```

### Run Tests

```bash
# Backend
cd backend && npm test

# Worker
cd worker && pytest tests/ -v
```

## Kubernetes (local k3d/kind)

```bash
# Create cluster
k3d cluster create atp --port "80:80@loadbalancer"

# Apply manifests (staging overlay)
kubectl apply -k k8s/overlays/staging

# Check rollout
kubectl rollout status deploy/backend -n ai-task-platform
```

## Argo CD Setup

```bash
# Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Apply project + applications
kubectl apply -f argocd/project.yaml
kubectl apply -f argocd/application-staging.yaml
kubectl apply -f argocd/application-production.yaml

# Get admin password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

# Port-forward UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## GitHub Actions Secrets Required

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `INFRA_REPO_PAT` | GitHub PAT with write access to infra repo |

## API Reference

### Auth
| Method | Path | Description |
|---|---|---|
| POST | `/api/auth/register` | Register new user |
| POST | `/api/auth/login` | Login, receive JWT |
| GET | `/api/auth/profile` | Get current user (auth required) |

### Tasks
| Method | Path | Description |
|---|---|---|
| POST | `/api/tasks` | Create task |
| GET | `/api/tasks` | List user's tasks |
| GET | `/api/tasks/:id` | Get task details |
| POST | `/api/tasks/:id/run` | Enqueue task for processing |
| DELETE | `/api/tasks/:id` | Delete task |

### Operations
- `uppercase` — converts input to UPPERCASE
- `lowercase` — converts input to lowercase
- `reverse_string` — reverses the string
- `word_count` — returns the word count as a string

## Environment Variables

### Backend / Worker shared
```
MONGODB_URI      MongoDB connection string
REDIS_HOST       Redis host
REDIS_PORT       Redis port (default: 6379)
REDIS_PASSWORD   Redis password (optional)
```

### Backend only
```
JWT_SECRET       JWT signing secret (min 32 chars)
JWT_EXPIRY       Token expiry (default: 7d)
PORT             HTTP port (default: 5000)
NODE_ENV         development | production
CORS_ORIGIN      Comma-separated allowed origins
LOG_LEVEL        info | debug | warn
```

### Frontend
```
VITE_API_URL     Backend API base URL
```

## Assumptions

1. BullMQ (Node.js) and Python worker communicate via Redis using BullMQ's internal list structure (`bull:{queue}:wait`). The Python side uses a direct `BRPOPLPUSH` approach rather than the full BullMQ protocol, which is sufficient for this workload.
2. MongoDB runs without authentication in the local/dev setup. Add `--auth` and credentials for production.
3. The Kubernetes `Secret` manifests contain placeholder base64 values — replace with real values via `kubectl create secret` or a secrets manager (Vault, Sealed Secrets) before production deployment.
4. Argo CD `sourceRepos` and `repoURL` values use placeholder `YOUR_ORG` — replace with actual GitHub org/repo names.



Project Structure
✔ Namespace
✔ ConfigMap
✔ Secret
✔ MongoDB Deployment
✔ Redis Deployment
✔ Backend Deployment
✔ Frontend Deployment
✔ Worker Deployment
✔ Ingress
✔ HPA
