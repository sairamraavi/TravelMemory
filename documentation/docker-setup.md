# Docker Setup Documentation - TravelMemory Application

## Project Overview

TravelMemory is a full-stack MERN application consisting of:
- **Frontend**: React application served by Nginx
- **Backend**: Node.js/Express API server
- **Database**: MongoDB for data persistence

This documentation covers the complete Docker setup process to run all services locally using Docker Compose.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Architecture Overview](#architecture-overview)
3. [Git Configuration](#git-configuration)
4. [Project Structure](#project-structure)
5. [Step-by-Step Setup](#step-by-step-setup)
6. [Running the Application](#running-the-application)
7. [Verifying Services](#verifying-services)
8. [Accessing the Application](#accessing-the-application)
9. [Troubleshooting](#troubleshooting)
10. [Cleanup](#cleanup)

---

## Prerequisites

Before starting, ensure you have installed:
- **Docker**: [Download Docker Desktop](https://www.docker.com/products/docker-desktop)
- **Docker Compose**: Included with Docker Desktop
- **Git**: For version control
- **Node.js & npm**: For local development (optional, not needed for Docker setup)

### Verify Installation

```bash
docker --version
docker compose version
git --version
```

---

## Architecture Overview

### Service Communication

```
┌─────────────────────────────────────────────────────────┐
│                   Local Machine                          │
│                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   Frontend   │  │   Backend    │  │   MongoDB    │   │
│  │ (Nginx:80)   │  │  (Node:3001) │  │ (Mongo:27017)│   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│       ▲                   ▲                   ▲           │
│       │                   │                   │           │
│       └───────────────────┼───────────────────┘           │
│         b16a_tm_network (Docker Bridge)                   │
│                                                           │
│  Exposed Ports:                                           │
│  - Frontend: http://localhost:80                         │
│  - Backend: http://localhost:3001                        │
│  - MongoDB: localhost:27019                              │
└─────────────────────────────────────────────────────────┘
```

### Service Details

| Service | Image | Port | Container Name | Network |
|---------|-------|------|-----------------|---------|
| Frontend | nginx:latest | 80:80 | tm_frontend | b16a_tm_network |
| Backend | node:20-alpine | 3001:3001 | tm_backend | b16a_tm_network |
| MongoDB | mongo:latest | 27019:27017 | tm_database | b16a_tm_network |

---

## Git Configuration

### Step 1: Set Git User Configuration

Configure your Git identity (required for commits):

```bash
# Set global Git user name
git config --global user.name "Your Name"

# Set global Git user email
git config --global user.email "your.email@example.com"

# Verify configuration
git config --global --list
```

### Step 2: Check Current Branch

```bash
cd /Users/sairaavi/Documents/MyLearnings/DevOps/projects/MERN/TravelMemory

git branch
# Output: * main
#           feature/docker-setup-config
```

### Step 3: Verify Remote Repository

```bash
git remote -v
# Output:
# origin  https://github.com/sairamraavi/TravelMemory.git (fetch)
# origin  https://github.com/sairamraavi/TravelMemory.git (push)
```

---

## Project Structure

```
TravelMemory/
├── .git/                          # Git configuration
├── docker-compose.yml             # Docker Compose configuration
├── .gitignore                     # Git ignore rules
├── README.md                      # Project README
├── LICENSE                        # License file
├── azure-pipelines.yml            # Azure CI/CD pipeline
│
├── backend/                       # Node.js/Express backend
│   ├── Dockerfile                # Backend container configuration
│   ├── .dockerignore              # Excluded files from Docker build
│   ├── package.json              # NPM dependencies
│   ├── .env                      # Environment variables
│   ├── index.js                  # Entry point
│   ├── conn.js                   # MongoDB connection
│   ├── controllers/              # API controllers
│   ├── models/                   # Database models
│   └── routes/                   # API routes
│
├── frontend/                      # React frontend
│   ├── Dockerfile                # Frontend container configuration
│   ├── .dockerignore              # Excluded files from Docker build
│   ├── nginx.conf                # Nginx configuration
│   ├── package.json              # NPM dependencies
│   ├── public/                   # Static files
│   ├── src/                      # React source code
│   └── build/                    # Production build (generated)
│
└── documentation/                # Documentation
    └── docker-setup.md           # This file
```

---

## Step-by-Step Setup

### Step 1: Navigate to Project Directory

```bash
cd /Users/sairaavi/Documents/MyLearnings/DevOps/projects/MERN/TravelMemory
```

### Step 2: Verify Project Structure

```bash
ls -la

# Expected output:
# -rw-r--r--  docker-compose.yml
# drwxr-xr-x  backend/
# drwxr-xr-x  frontend/
# drwxr-xr-x  documentation/
# -rw-r--r--  README.md
```

### Step 3: Review Configuration Files

#### a) docker-compose.yml

```bash
cat docker-compose.yml
```

**Key components:**
- `frontend`: React app served by nginx on port 80
- `backend`: Node.js API on port 3001
- `database`: MongoDB on port 27017 (exposed as 27019)
- `networks`: All services connected via `b16a_tm_network`

#### b) Backend Dockerfile

```bash
cat backend/Dockerfile
```

**Configuration:**
- Base image: `node:20-alpine` (lightweight, secure)
- Installs dependencies: `npm ci` (deterministic installation)
- Exposes port: 3001
- Command: `node index.js`

#### c) Frontend Dockerfile

```bash
cat frontend/Dockerfile
```

**Configuration:**
- Build stage: `node:20-alpine` compiles React app
- Production stage: `nginx:1.26.3-alpine` serves static files
- Exposes port: 80

#### d) Nginx Configuration

```bash
cat frontend/nginx.conf
```

**Key features:**
- Upstream backend server: `backend:3001`
- Proxy routes: `/trip`, `/hello` → backend service
- React routing: `try_files` fallback to `index.html`

#### e) Backend Environment Variables

```bash
cat backend/.env
```

**Variables:**
- `MONGO_URI`: MongoDB connection string (overridden by docker-compose)
- `PORT`: Server port (overridden by docker-compose)

### Step 4: Review Backend Connection Configuration

```bash
cat backend/conn.js
```

**Connection logic:**
- Uses `process.env.MONGO_URI` from docker-compose
- Connects to MongoDB service using hostname: `tm_database`

### Step 5: Review Frontend URL Configuration

```bash
cat frontend/src/url.js
```

**Configuration:**
- Base URL defaults to relative path (empty string)
- Proxied through nginx to backend service

---

## Running the Application

### Step 1: Build and Start All Services

```bash
# Make sure you're in the project root directory
cd /Users/sairaavi/Documents/MyLearnings/DevOps/projects/MERN/TravelMemory

# Build Docker images and start containers in detached mode
docker compose up -d

# Expected output:
# [+] Running 4/4
#  ✔ Network b16a_tm_network Created
#  ✔ Container tm_database Created
#  ✔ Container tm_backend Created
#  ✔ Container tm_frontend Created
```

### Step 2: Wait for Services to Start

Wait approximately 15-30 seconds for all services to initialize:
- MongoDB: Starts and initializes database
- Backend: Connects to MongoDB
- Frontend: Builds React app and starts nginx

### Step 3: Verify Container Status

```bash
docker ps

# Expected output:
# CONTAINER ID   IMAGE                      COMMAND                 STATUS          PORTS
# xxxxxxxx       travelmemory-frontend      "nginx -g daemon off"   Up 2 minutes    0.0.0.0:80->80/tcp
# xxxxxxxx       travelmemory-backend       "node index.js"         Up 3 minutes    0.0.0.0:3001->3001/tcp
# xxxxxxxx       mongo:latest               "docker-entrypoint.s…"  Up 4 minutes    0.0.0.0:27019->27017/tcp
```

---

## Verifying Services

### Verification Step 1: Check Container Logs

#### Frontend Logs

```bash
docker compose logs tm_frontend

# Expected output:
# tm_frontend  | 2026-05-03 10:00:00 UTC [notice] Nginx process started...
# tm_frontend  | [info] 1#1: signal process started
```

#### Backend Logs

```bash
docker compose logs tm_backend

# Expected output:
# tm_backend  | Server started at http://localhost:3001
# tm_backend  | (node:1) Warning: ...
```

#### MongoDB Logs

```bash
docker compose logs tm_database

# Expected output:
# tm_database  | initramfs: Ready to accept connections
```

### Verification Step 2: Test Backend API

#### Test Basic Endpoint

```bash
curl http://localhost:3001/hello

# Expected response:
# Hello World!
```

#### Test Trip Endpoint (GET)

```bash
curl http://localhost:3001/trip

# Expected response:
# [] (empty array initially)
```

### Verification Step 3: Test Frontend Health

```bash
curl http://localhost

# Expected response:
# <html> ... (HTML content from React build)
```

### Verification Step 4: Check Network Connectivity

```bash
# List all networks
docker network ls

# Inspect the project network
docker network inspect b16a_tm_network

# Expected output shows all 3 containers connected
```

---

## Accessing the Application

### Frontend Access

```
URL: http://localhost
Browser: Open http://localhost in any web browser
Port: 80
```

**What you'll see:**
- React application homepage
- Travel Memory UI components
- Navigation to add experiences and view details

### Backend API Access

```
Base URL: http://localhost:3001
Available Endpoints:
  - GET  http://localhost:3001/hello
  - GET  http://localhost:3001/trip
  - POST http://localhost:3001/trip
```

### Database Access

#### Option 1: Direct MongoDB Connection

```bash
# Connect to MongoDB from host machine
mongosh --host localhost --port 27019

# Inside mongosh:
use travelmemory
db.trips.find()
```

#### Option 2: From Inside Backend Container

```bash
# Execute command inside backend container
docker exec -it tm_backend mongosh mongodb://tm_database:27017/travelmemory
```

---

## Troubleshooting

### Issue 1: Frontend Container Not Starting

**Error Message:**
```
nginx: [emerg] "events" directive is not allowed here
```

**Solution:**
The nginx.conf file has invalid syntax. Ensure it only contains `upstream` and `server` blocks, NOT `events` or `http` blocks.

**Check:**
```bash
cat frontend/nginx.conf | head -5
# Should start with: upstream backend or server, NOT events
```

### Issue 2: Backend Cannot Connect to MongoDB

**Error Message:**
```
MongoServerError: connect ENOTFOUND tm_database
```

**Solution:**
Services must be on the same Docker network. Verify:

```bash
# Check docker-compose backend environment
docker compose config | grep -A 10 "backend:"

# Should show: MONGO_URI: "mongodb://tm_database:27017/travelmemory"
```

### Issue 3: Frontend Cannot Connect to Backend

**Error Message:**
```
Failed to fetch from backend
```

**Solution:**
Check nginx proxy configuration:

```bash
curl -v http://localhost/trip

# Should proxy to backend:3001
```

### Issue 4: Port Already in Use

**Error Message:**
```
bind: address already in use
```

**Solution:**
Kill existing services and restart:

```bash
# Stop all containers
docker compose down

# Remove all containers and volumes (if needed)
docker compose down -v

# Restart
docker compose up -d
```

### Issue 5: Image Pull Failures

**Error Message:**
```
failed to resolve reference "mongo:7-alpine": docker.io/library/mongo:7-alpine: not found
```

**Solution:**
Use `mongo:latest` instead. Already fixed in docker-compose.yml.

### Debug Mode: View Live Logs

```bash
# Follow all service logs in real-time
docker compose logs -f

# Follow specific service logs
docker compose logs -f tm_backend
docker compose logs -f tm_frontend
docker compose logs -f tm_database
```

---

## Cleanup

### Stop All Services

```bash
# Stop containers (keeps data)
docker compose down

# Expected output:
# [+] Stopping 3/3
#  ✔ Container tm_frontend Stopped
#  ✔ Container tm_backend Stopped
#  ✔ Container tm_database Stopped
#  ✔ Network b16a_tm_network Removed
```

### Remove All Services and Data

```bash
# Remove containers, networks, and volumes
docker compose down -v

# This will:
# - Stop all containers
# - Remove containers
# - Remove network
# - Remove MongoDB data volume
```

### Remove Docker Images

```bash
# View all images
docker images | grep travelmemory

# Remove images
docker rmi travelmemory-frontend travelmemory-backend
```

---

## Useful Docker Commands

### View Running Containers

```bash
docker ps                          # Running containers only
docker ps -a                       # All containers (running + stopped)
```

### View Container Details

```bash
docker inspect tm_backend          # Full container configuration
docker stats                       # Real-time resource usage
```

### Execute Commands Inside Container

```bash
docker exec -it tm_backend sh      # Interactive shell in backend
docker exec tm_backend npm --version
docker exec -it tm_database mongosh
```

### View Environment Variables

```bash
docker exec tm_backend env | grep -E "(PORT|MONGO)"
```

### Restart Services

```bash
docker compose restart              # Restart all services
docker compose restart tm_backend   # Restart specific service
```

---

## Summary

This Docker setup enables:

✅ **Local Development**: All services run in containers matching production environment
✅ **Easy Deployment**: Single command (`docker compose up`) to start entire stack
✅ **Data Persistence**: MongoDB data stored in Docker volume
✅ **Service Communication**: All services connected via Docker network
✅ **Port Isolation**: Services accessible on specific ports
✅ **Scalability**: Easy to add more services or scale existing ones

### Quick Reference

```bash
# Navigate to project
cd /Users/sairaavi/Documents/MyLearnings/DevOps/projects/MERN/TravelMemory

# Start services
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down

# Access application
# Frontend: http://localhost
# Backend: http://localhost:3001
# Database: localhost:27019
```

---

## Next Steps

1. ✅ Complete: Docker setup and documentation
2. → Git Feature Branch: `feature/docker-setup-config`
3. → Test all endpoints manually
4. → Create pull request to `main` branch
5. → Review and merge changes

---

## Support

For issues or questions:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review Docker Compose logs: `docker compose logs`
3. Check Docker desktop application for additional details
4. Consult Docker documentation: https://docs.docker.com/

---

**Last Updated**: May 3, 2026
**Branch**: feature/docker-setup-config
**Status**: Complete
