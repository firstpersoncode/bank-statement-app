# Bank Statement Application

A full-stack monorepo application for uploading and analyzing bank statement CSV files. The application consists of a Go backend API and a Next.js frontend, both containerized with Docker for easy deployment.

## Monorepo Structure

```
bank-statement-app/
├── go-uploader/          # Backend service (Go + Fiber)
├── next-uploader/        # Frontend service (Next.js + React)
├── docker-compose.yml    # Multi-service orchestration
├── .gitignore           # Root gitignore for monorepo
└── README.md            # This file
```

## Architecture

- **Backend**: Go (Fiber framework) - RESTful API for transaction management and authentication
- **Frontend**: Next.js (React 19) - Modern web interface with TypeScript
- **Deployment**: Docker & Docker Compose - Containerized services for consistent environments
- **Repository**: Monorepo structure with unified version control

## Quick Start

### Prerequisites

- Docker Desktop installed and running
- Git

### Installation

1. **Clone the repository**
   ```bash
   git clone --recurse-submodules <repository-url>
   cd bank-statement-app
   ```

2. **Configure environment variables**

   Create `.env` file in `go-uploader/`:
   ```bash
   cd go-uploader
   cp .env.example .env
   ```

   Edit `go-uploader/.env`:
   ```env
   SESSION_COOKIE_NAME=__session__
   HOST=0.0.0.0
   PORT=8080
   ALLOWED_ORIGINS=http://localhost:3000
   ```

   **Important:** Set `HOST=0.0.0.0` (not `localhost`) to make the backend accessible from outside the container.

   Create `.env` file in `next-uploader/`:
   ```bash
   cd ../next-uploader
   cp .env.example .env
   ```

   Edit `next-uploader/.env`:
   ```env
   NEXT_PUBLIC_BASE_API_URL=http://localhost:8080
   ```

3. **Build and run containers**
   ```bash
   cd ..
   docker compose up --build -d
   ```

   This will:
   - Build the Go backend container
   - Build the Next.js frontend container
   - Start both services in detached mode
   - Set up networking between containers

4. **Access the application**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8080

### Docker Commands

```bash
# Build containers
docker compose build

# Start containers (detached)
docker compose up -d

# Start containers (with logs)
docker compose up

# Stop containers
docker compose down

# View logs
docker compose logs

# View logs for specific service
docker compose logs backend
docker compose logs frontend

# Restart a service
docker compose restart backend

# Rebuild and restart
docker compose up --build -d
```

## Container Details

### Backend Container (`go-uploader`)

**Base Image**: `golang:1.24-alpine` (build), `alpine:latest` (runtime)

**Features**:
- Multi-stage build for minimal image size
- Health check endpoint: `/health`
- Exposed port: `8080`
- Auto-restart on failure

**Health Check**:
```bash
# Check container health
docker compose ps

# Manual health check
curl http://localhost:8080/health
```

### Frontend Container (`next-uploader`)

**Base Image**: `node:20-alpine`

**Features**:
- Multi-stage build for optimized production bundle
- Production-only dependencies in runtime
- Exposed port: `3000`
- Waits for backend to be healthy before starting

**Build Process**:
1. Install all dependencies
2. Build Next.js application
3. Copy production files to runtime image
4. Install only production dependencies

## Troubleshooting

### Backend not accessible at localhost:8080

**Problem**: Backend shows as running but cannot connect from host machine.

**Solution**: Ensure `HOST=0.0.0.0` in `go-uploader/.env` (not `localhost` or `127.0.0.1`)

```env
# ✗ Wrong
HOST=localhost

# ✓ Correct
HOST=0.0.0.0
```

Then restart:
```bash
docker compose restart backend
```

### Container build fails

**Check Docker Desktop is running**:
```bash
docker ps
```

**View build logs**:
```bash
docker compose build --no-cache
```

**Clean and rebuild**:
```bash
docker compose down
docker compose build --no-cache
docker compose up -d
```

### Frontend cannot connect to backend

1. **Check backend is healthy**:
   ```bash
   docker compose ps
   curl http://localhost:8080/health
   ```

2. **Check frontend environment variable**:
   ```env
   # next-uploader/.env
   NEXT_PUBLIC_BASE_API_URL=http://localhost:8080
   ```

3. **Restart frontend**:
   ```bash
   docker compose restart frontend
   ```

### View container logs

```bash
# All services
docker compose logs

# Specific service
docker compose logs backend

# Follow logs (real-time)
docker compose logs -f

# Last 100 lines
docker compose logs --tail=100
```

### Container keeps restarting

**Check logs for errors**:
```bash
docker compose logs backend
```

**Common issues**:
- Missing `.env` files
- Invalid environment variable values
- Port conflicts (8080 or 3000 already in use)

**Check port conflicts**:
```bash
# macOS/Linux
lsof -i :8080
lsof -i :3000

# Kill process using port
kill -9 <PID>
```

## Project Documentation

For detailed information about each service:

- **Backend Documentation**: [go-uploader/README.md](https://github.com/firstpersoncode/go-uploader/blob/master/README.md)
  - API endpoints and usage
  - Authentication flow
  - Architecture decisions
  - Development setup

- **Frontend Documentation**: [next-uploader/README.md](https://github.com/firstpersoncode/next-uploader/blob/main/README.md)
  - Component architecture
  - State management
  - API integration
  - Development setup

## Development Without Docker

If you prefer to run services locally without Docker:

### Backend
```bash
cd go-uploader
go mod download
go run main.go
```

### Frontend
```bash
cd next-uploader
npm install
npm run dev
```

See individual README files for detailed local development instructions.

## Monorepo Management

### Repository Structure

This is a **monorepo** using **git submodules** to manage both frontend and backend services. Each service:
- Maintains its own git history and repository
- Has independent dependencies (`go.mod` for backend, `package.json` for frontend)
- Can be developed and versioned separately
- Is linked to the monorepo for unified orchestration

### Git Submodules Workflow

**Initial Clone:**
```bash
# Clone the monorepo with submodules
git clone --recurse-submodules <repository-url>
cd bank-statement-app

# Or if already cloned without submodules:
git submodule update --init --recursive
```

**Working with Submodules:**
```bash
# Check submodule status
git submodule status

# Update submodules to latest commits
git submodule update --remote

# Make changes in a submodule
cd go-uploader
git checkout -b feature/my-feature
# Make changes...
git add .
git commit -m "feat: add new feature"
git push origin feature/my-feature

# Update monorepo to reference new commit
cd ..
git add go-uploader
git commit -m "chore: update go-uploader submodule"
git push
```

**Benefits:**
- **Preserved History**: Full commit history for each service
- **Independent Development**: Each service has its own repository
- **Unified Orchestration**: Single docker-compose for both services
- **Flexible Versioning**: Pin specific versions of each service

### Working with Services

Each service maintains its own git repository:

```bash
# Work on backend (has its own git repo)
cd go-uploader
git status  # Shows go-uploader repo status
go run main.go

# Work on frontend (has its own git repo)
cd next-uploader
git status  # Shows next-uploader repo status
npm run dev
```

### Submodule Repositories

The submodules link to these repositories:
- **Backend**: https://github.com/firstpersoncode/go-uploader
- **Frontend**: https://github.com/firstpersoncode/next-uploader

Changes made in the submodules can be pushed directly to these repositories.

---
