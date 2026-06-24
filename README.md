# Smart Mirror Docker Setup

A full-stack smart mirror services with:
- **API Gateway** (Traefik) — routes `/api/v1/*` and `/api/v1/ai/*` on `http://localhost:3000`
- **Face Recognition Service** (Python/AI)
- **Core API** (Node.js)
- **Databases** — MongoDB, Redis, Qdrant
- **Logging** (Dozzle)

---

## Quick Links

- [Prerequisites — Install Docker](#prerequisites--install-docker)
- [Check & Free Required Ports](#check--free-required-ports)
- [Setup Environment Variables](#setup-environment-variables)
- [Run the Application](#run-the-application)
- [Access the Services](#access-the-services)
- [Useful Commands](#useful-commands)

---

## Prerequisites — Install Docker

### Windows

1. Download **[Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)**.
2. Run the installer and follow the prompts (WSL 2 backend is recommended).
3. After installation, launch Docker Desktop from the Start menu.
4. Wait for the Docker whale icon to appear in the system tray (status: *Running*).

### Linux (Ubuntu / Debian)

```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y ca-certificates curl gnupg

# Add Docker's official GPG key and repository
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine & Compose plugin
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add your user to the docker group (so you can run docker without sudo)
sudo usermod -aG docker $USER
newgrp docker
```

Verify the installation:

```bash
docker --version && docker compose version
```

---

## Check & Free Required Ports

The application uses these ports on your host machine:

| Port | Service        |
|------|----------------|
| 80   | Dashboard      |
| 3000 | API Gateway    |
| 6333 | Qdrant         |
| 6379 | Redis          |
| 8080 | Dozzle (logs)  |
| 9000 | Traefik UI     |
| 27017| MongoDB        |

### Windows (PowerShell)

Check if a port is in use:

```powershell
netstat -ano | findstr ":80 "
netstat -ano | findstr ":3000 "
```

If a port is occupied, find the process ID (PID, last column) and stop it:

```powershell
# Find the PID using the port (replace PORT with the actual number)
netstat -ano | findstr ":PORT "

# Stop the process (replace PID with the actual number)
Stop-Process -Id PID -Force
```

Common offenders:
- **Port 80** — often used by IIS or World Wide Web Publishing Service. Stop it with:
  ```powershell
  Stop-Service -Name W3SVC -Force
  Set-Service -Name W3SVC -StartupType Disabled
  ```
- **Port 3000** — may be used by another Node.js process.

### Linux

```bash
# Check which ports are in use
sudo ss -tuln | grep -E ':(80|3000|6333|6379|8080|9000|27017) '
```

Kill a process using a specific port:

```bash
# Find the PID (replace PORT)
sudo lsof -i :PORT

# Kill the process (replace PID)
sudo kill -9 PID
```

Alternatively, stop any service listening on port 80:

```bash
sudo systemctl stop nginx     # if using Nginx
sudo systemctl stop apache2   # if using Apache
```

---

## Setup Environment Variables

A pre-configured `.env` file is already provided in this directory. It contains all the necessary variables for local development.

If you need to regenerate it from scratch, create a file named `.env` in this folder with the following content (update secrets accordingly):

```env
DEBUG=True

# Qdrant
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=
QDRANT_COLLECTION_NAME=mrayti_faces

# Redis
REDIS_URL=redis://localhost:6379/0
REDIS_STREAM_NAME=mrayti_events
REDIS_CONSUMER_GROUP=ai_service_group

# Limits
MAX_IMAGE_SIZE_BYTES=5242880

# Rate Limiting
AUTH_RATE_LIMIT=5 per minute
REGISTRATION_RATE_LIMIT=10 per minute

# Architecture
BEHIND_GATEWAY=True

# Environment
ENVIRONMENT=development

# Node.js API
PORT=3000
MONGO_URI=mongodb://smart-mirror-mongo:27017/smart_mirror
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES=7d
GEMINI_API_KEY=your_gemini_api_key_here
```

> **Note:** The `MONGO_URI` uses the Docker service name `smart-mirror-mongo` as the host — this works because all containers are on the same Docker network.

---

## Run the Application

Make sure your terminal is in the same directory as `docker-compose.yaml`, then run:

```bash
docker compose up -d
```

- `-d` — runs containers in the background (detached mode).

Docker will download the required images and start all services. This may take a few minutes the first time.

### Stop the Application

```bash
docker compose down
```

---

## Access the Services

| Service               | URL                                   |
|-----------------------|---------------------------------------|
| API Gateway (Traefik) | http://localhost:3000                 |
| API Routes            | http://localhost:3000/api/v1/...      |
| AI Routes             | http://localhost:3000/api/v1/ai/...   |
| Traefik Dashboard     | http://localhost:9000                 |
| Dozzle (Log Viewer)   | http://localhost:8080                 |
| MongoDB               | localhost:27017                       |
| Redis                 | localhost:6379                        |
| Qdrant Dashboard      | localhost:6333/dashboard              |

---

## Useful Commands

```bash
# List running services
docker compose ps

# View live logs from all services
docker compose logs -f

# View logs for a specific service
docker compose logs -f traefik

# Restart all services
docker compose restart

# Stop and remove containers (data volumes are preserved)
docker compose down

# Stop and remove everything including volumes
docker compose down -v

# Rebuild and start (if you made local changes)
docker compose up -d --build
```
