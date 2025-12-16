# NoTap Backend - Local Development Setup Guide

**Last Updated:** 2025-12-10
**Estimated Setup Time:** 10-15 minutes

---

## Overview

This guide helps you set up the complete NoTap backend infrastructure locally for **FREE** using Docker. You'll have:

- **PostgreSQL 15** - Database for persistent storage
- **Redis 7** - Caching and session management
- **Node.js Backend** - API server
- **Optional GUI tools** - pgAdmin and Redis Commander

---

## Prerequisites

Before starting, ensure you have:

| Requirement | Version | Download Link |
|-------------|---------|---------------|
| **Docker Desktop** | Latest | [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) |
| **Node.js** | 18+ | [nodejs.org](https://nodejs.org/) |
| **npm** | 9+ | Included with Node.js |
| **Git** | Latest | [git-scm.com](https://git-scm.com/) |

### Verify Prerequisites

```bash
# Check Docker
docker --version
# Docker version 24.x.x

# Check Docker Compose
docker compose version
# Docker Compose version v2.x.x

# Check Node.js
node -v
# v20.x.x (or v18.x.x)

# Check npm
npm -v
# 10.x.x (or 9.x.x)
```

---

## Quick Start (5 minutes)

### Option A: Automated Setup (Recommended)

**Linux/macOS/WSL:**
```bash
cd backend
chmod +x scripts/local-setup.sh
./scripts/local-setup.sh
```

**Windows PowerShell:**
```powershell
cd backend
.\scripts\local-setup.ps1
```

### Option B: Manual Setup

```bash
# 1. Navigate to backend
cd backend

# 2. Create environment file
cp .env.local.example .env.local

# 3. Install dependencies
npm install

# 4. Start Docker containers
docker compose up -d postgres redis

# 5. Wait for services (30 seconds)
sleep 30

# 6. Start the backend
npm run dev
```

---

## What Gets Installed

### Docker Containers

| Service | Container Name | Port | Purpose |
|---------|---------------|------|---------|
| PostgreSQL | notap-postgres | 5432 | Main database |
| Redis | notap-redis | 6379 | Cache & sessions |
| pgAdmin | notap-pgadmin | 5050 | DB GUI (optional) |
| Redis Commander | notap-redis-commander | 8081 | Redis GUI (optional) |

### Default Credentials (Development Only)

| Service | Username | Password | Database |
|---------|----------|----------|----------|
| PostgreSQL | `notap` | `notap_dev_password_2024` | `notap_dev` |
| Redis | - | `notap_redis_dev_2024` | - |
| pgAdmin | `admin@notap.local` | `admin123` | - |

⚠️ **NEVER use these credentials in production!**

---

## Detailed Commands

### Start Services

```bash
# Start PostgreSQL and Redis only
docker compose up -d postgres redis

# Start all services including backend
docker compose up -d

# Start with optional GUI tools
docker compose --profile tools up -d
```

### Stop Services

```bash
# Stop all containers
docker compose down

# Stop and remove all data (full reset)
docker compose down -v
```

### View Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f postgres
docker compose logs -f redis
docker compose logs -f backend
```

### Check Status

```bash
# Container status
docker compose ps

# PostgreSQL health
docker exec notap-postgres pg_isready -U notap

# Redis health
docker exec notap-redis redis-cli -a notap_redis_dev_2024 ping
```

---

## Running the Backend

### Development Mode (with hot reload)

```bash
cd backend
npm run dev
```

Output:
```
🚀 NoTap Backend running on port 3000
📦 Redis connected
🗄️ PostgreSQL connected
```

### Production Mode

```bash
npm start
```

---

## Running Tests

### All Tests

```bash
npm run test:all
```

### Individual Test Suites

```bash
# Unit tests
npm test

# Crypto tests
npm run test:crypto

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e
```

### Bugster E2E Tests

```bash
# Backend API tests
bugster run --config bugster.config.yaml

# Web UI tests
bugster run --config bugster.config.web.yaml
```

---

## API Endpoints

Once running, the API is available at: **http://localhost:3000**

### Health Check

```bash
curl http://localhost:3000/health
```

Response:
```json
{
  "status": "healthy",
  "timestamp": "2025-12-10T01:00:00.000Z",
  "services": {
    "redis": "connected",
    "postgres": "connected"
  }
}
```

### API Documentation

| Endpoint | Description |
|----------|-------------|
| `GET /health` | Health check |
| `POST /v1/enrollment` | Create enrollment |
| `GET /v1/enrollment/:uuid` | Get enrollment |
| `POST /v1/verification/initiate` | Start verification |
| `POST /v1/verification/verify` | Verify factors |

See `backend/docs/` for complete API documentation.

---

## Database Management

### Connect to PostgreSQL

```bash
# Using Docker
docker exec -it notap-postgres psql -U notap -d notap_dev

# Using psql locally
psql postgresql://notap:notap_dev_password_2024@localhost:5432/notap_dev
```

### Run Migrations

```bash
npm run db:migrate
```

### Seed Database

```bash
npm run db:seed
```

### View Tables

```sql
-- Connect first, then:
\dt
```

---

## Redis Management

### Connect to Redis

```bash
# Using Docker
docker exec -it notap-redis redis-cli -a notap_redis_dev_2024

# Test connection
PING
# Response: PONG
```

### Common Commands

```bash
# List all keys
KEYS *

# Get key value
GET enrollment:uuid:abc123

# Check TTL
TTL enrollment:uuid:abc123

# Clear all (careful!)
FLUSHALL
```

---

## GUI Tools (Optional)

### pgAdmin (Database GUI)

```bash
# Start pgAdmin
docker compose --profile tools up -d pgadmin
```

Access: **http://localhost:5050**
- Email: `admin@notap.local`
- Password: `admin123`

To connect to PostgreSQL:
- Host: `postgres` (Docker network name)
- Port: `5432`
- Username: `notap`
- Password: `notap_dev_password_2024`
- Database: `notap_dev`

### Redis Commander (Redis GUI)

```bash
# Start Redis Commander
docker compose --profile tools up -d redis-commander
```

Access: **http://localhost:8081**

---

## Troubleshooting

### Docker Issues

**"Cannot connect to Docker daemon"**
```bash
# Start Docker Desktop, then retry
# On Linux, ensure user is in docker group:
sudo usermod -aG docker $USER
newgrp docker
```

**"Port already in use"**
```bash
# Find what's using the port
lsof -i :5432  # PostgreSQL
lsof -i :6379  # Redis
lsof -i :3000  # Backend

# Kill the process or change ports in docker-compose.yml
```

### PostgreSQL Issues

**"Connection refused"**
```bash
# Check if container is running
docker ps | grep postgres

# Check logs
docker logs notap-postgres

# Restart container
docker compose restart postgres
```

### Redis Issues

**"NOAUTH Authentication required"**
```bash
# Use password in commands
redis-cli -a notap_redis_dev_2024 PING
```

### Backend Issues

**"Cannot find module"**
```bash
# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

**"ECONNREFUSED to Redis/PostgreSQL"**
```bash
# Ensure containers are running
docker compose ps

# Check container health
docker compose logs postgres
docker compose logs redis
```

---

## Environment Variables

Key variables in `.env.local`:

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | development | Environment mode |
| `PORT` | 3000 | API port |
| `DATABASE_URL` | (see file) | PostgreSQL connection |
| `REDIS_HOST` | localhost | Redis host |
| `REDIS_PORT` | 6379 | Redis port |
| `REDIS_PASSWORD` | (see file) | Redis password |
| `ENCRYPTION_KEY` | (see file) | AES-256 key |

For production variables, see `backend/.env.example`.

---

## Switching to Production

When ready for production:

1. **Get real credentials** for PostgreSQL and Redis
2. **Copy `.env.example`** to `.env` and fill in real values
3. **Enable TLS** for Redis
4. **Use AWS KMS** for encryption
5. **Set up proper monitoring**

See `documentation/06-deployment/` for production guides.

---

## Quick Reference

```bash
# === SETUP ===
./scripts/local-setup.sh          # Full automated setup
docker compose up -d              # Start services

# === DEVELOPMENT ===
npm run dev                       # Start backend (hot reload)
npm test                          # Run tests

# === DATABASE ===
npm run db:migrate                # Run migrations
docker exec -it notap-postgres psql -U notap -d notap_dev

# === REDIS ===
docker exec -it notap-redis redis-cli -a notap_redis_dev_2024

# === LOGS ===
docker compose logs -f            # All logs
docker compose logs -f backend    # Backend only

# === CLEANUP ===
docker compose down               # Stop services
docker compose down -v            # Stop + delete data
./scripts/local-setup.sh --reset  # Full reset
```

---

## Next Steps

1. ✅ **Test the API** - `curl http://localhost:3000/health`
2. ✅ **Run tests** - `npm test`
3. ✅ **Explore endpoints** - See `backend/docs/`
4. ✅ **Check logs** - `docker compose logs -f`

For questions, see `documentation/02-user-guides/TROUBLESHOOTING_GUIDE.md`.
