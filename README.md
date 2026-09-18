# MEAN Stack CRUD Application - Containerization & Automated Cloud Deployment

A production-grade deployment and automation setup for a full-stack **MEAN** (MongoDB, Express, Angular 15, Node.js) CRUD application. This repository includes complete containerization with **Docker**, multi-service orchestration with **Docker Compose**, an **Nginx reverse proxy** built directly into the frontend container routing all traffic via standard HTTP (Port 80), and an automated **CI/CD pipeline** powered by **GitHub Actions**.

---

## 🏗️ Architecture Overview

```mermaid
flowchart TD
    subgraph Internet["Public Internet"]
        User["Client Browser\nhttp://&lt;VM_IP&gt;:80"]
    end

    subgraph VM["Ubuntu Cloud VM (AWS EC2 / Azure VM)"]
        subgraph DockerBridge["Docker Network: mean-network"]
            Frontend["Frontend Container (mean-frontend:80)\nAngular 15 Static Assets + Nginx Reverse Proxy"]
            Backend["Node.js Express API\n(mean-backend:8080)"]
            MongoDB[("MongoDB 6.0\n(mean-mongodb:27017)")]
            MongoVol[("Persistent Volume\nmongodb_data")]
        end
    end

    subgraph Automation["CI/CD Automation"]
        Dev["Developer Push\n(git push origin main)"]
        GHA["GitHub Actions Runner"]
        DockerHub[("Docker Hub Registry\nsomeshtarra/mean-*")]
    end

    User -->|"HTTP Port 80"| Frontend
    Frontend -->|"Static Assets (SPA fallback)"| Frontend
    Frontend -->|"Reverse Proxy: location /api/"| Backend
    Backend -->|"Mongoose connection (localhost:27017)"| MongoDB
    MongoDB --- MongoVol

    Dev --> GHA
    GHA -->|"1. Build & Push Images"| DockerHub
    GHA -->|"2. SSH Automated Deploy"| VM
```

---

## ✨ Features

- **Full-Stack CRUD Application**: Manage tutorials (create, retrieve, update, delete, search by title).
- **100% Untouched Application Code**: Application source code remains completely unchanged. Infrastructure, database connectivity, and CORS are handled strictly at the container and proxy layers.
- **Microservices Orchestration**: Docker Compose manages `mongodb`, `backend`, and `frontend`.
- **Integrated Nginx Reverse Proxy**: The frontend container runs Nginx, serving the Angular 15 SPA on `/` and reverse-proxying all REST API calls on `/api/` with edge CORS support.
- **Database Persistence**: MongoDB runs using the official `mongo:6.0` image with named volume persistence (`mongodb_data`).
- **Continuous Integration & Continuous Deployment (CI/CD)**: Push-to-deploy workflow with Docker Hub and automated remote SSH container updates.

---

## 📁 Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── ci-cd.yml          # GitHub Actions workflow (Build, Push & Deploy)
├── backend/
│   ├── app/
│   │   ├── config/            # DB configuration
│   │   ├── controllers/       # CRUD controller logic
│   │   ├── models/            # Mongoose schemas
│   │   └── routes/            # Express routes
│   ├── .dockerignore          # Backend build exclusions
│   ├── Dockerfile             # Production Node.js 18 Alpine image
│   ├── package.json
│   └── server.js              # Express app
├── frontend/
│   ├── src/                   # Angular 15 source code
│   ├── .dockerignore          # Frontend build exclusions
│   ├── Dockerfile             # Multi-stage build (Node builder + Nginx runner)
│   ├── nginx.conf             # Nginx reverse proxy & SPA routing configuration
│   ├── angular.json
│   └── package.json
├── docs/
│   └── screenshots/           # Deliverable screenshots and verification assets
├── .env.example               # Template environment configuration
├── docker-compose.yml         # Complete multi-container orchestration
└── README.md
```

---

## 🚀 Quickstart: Local Deployment with Docker Compose

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/) (v20.10+)
- [Docker Compose](https://docs.docker.com/compose/) (v2.0+)

### Steps

1. **Clone the repository**:
   ```bash
   git clone https://github.com/someshtarra/crud-dd-task-mean-app.git
   cd crud-dd-task-mean-app
   ```

2. **Configure environment variables**:
   ```bash
   cp .env.example .env
   ```

3. **Build and start all services**:
   ```bash
   docker compose up -d --build
   ```

4. **Verify container status**:
   ```bash
   docker compose ps
   ```
   All 3 containers (`mean-mongodb`, `mean-backend`, `mean-frontend`) should show status `Up` (healthy).

5. **Access the application**:
   - Web UI: Open [http://localhost](http://localhost) in your browser.
   - Backend API: [http://localhost/api/tutorials](http://localhost/api/tutorials) (routed through Nginx on Port 80) or [http://localhost:8080/api/tutorials](http://localhost:8080/api/tutorials)

6. **Stop services**:
   ```bash
   docker compose down
   ```
   *(To wipe database data, pass the `-v` flag: `docker compose down -v`)*

---

## ☁️ Cloud Deployment on Ubuntu Virtual Machine (AWS / Azure)

### 1. Provision an Ubuntu VM

- **Platform**: AWS EC2 or Azure Virtual Machines
- **OS**: Ubuntu 22.04 LTS or 24.04 LTS (x86_64)
- **Instance Size**: `t3.medium` or `t2.medium` (AWS) / `Standard_B2s` (Azure) recommended (minimum 2 vCPU, 4 GB RAM for Angular build and MongoDB runtime).

### 2. Configure Firewall / Security Group Rules

Allow the following inbound traffic in your Security Group (AWS) or Network Security Group (Azure):

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| `22` | TCP | Your IP / CI runner | SSH Remote Administration |
| `80` | TCP | `0.0.0.0/0` (Anywhere) | Public HTTP Application Access |

*(Port 27017 remains private inside the Docker network).*

### 3. Install Docker & Docker Compose on Ubuntu

SSH into your cloud VM:
```bash
ssh -i /path/to/key.pem ubuntu@<VM_PUBLIC_IP>
```

Run the automated Docker installation script:
```bash
# Update package repositories
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# Add Docker official GPG key & repo
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker engine and Compose plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Grant ubuntu user permission to run Docker without sudo
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker compose version
```

### 4. Deploy the Application on the VM

1. Create application directory:
   ```bash
   mkdir -p ~/mean-app && cd ~/mean-app
   ```

2. Copy `docker-compose.yml` into `~/mean-app/docker-compose.yml`.

3. Create the `.env` file:
   ```bash
   cat << 'EOF' > .env
   DOCKERHUB_USERNAME=someshtarra
   EOF
   ```

4. Pull the latest images from Docker Hub and launch:
   ```bash
   docker compose pull
   docker compose up -d
   ```

5. Check container logs:
   ```bash
   docker compose logs -f
   ```

---

## 🔄 CI/CD Pipeline Configuration (GitHub Actions)

The repository includes an automated workflow defined in [`.github/workflows/ci-cd.yml`](.github/workflows/ci-cd.yml).

### Pipeline Stages

1. **Build & Push (`build-and-push`)**:
   - Authenticates with Docker Hub.
   - Builds production images with multi-platform caching.
   - Pushes images with `latest` and commit SHA tags:
     - `<DOCKERHUB_USERNAME>/mean-backend`
     - `<DOCKERHUB_USERNAME>/mean-frontend`
2. **Automated Deploy (`deploy`)**:
   - Secures SSH connection into the Ubuntu cloud VM.
   - Transmits the latest `docker-compose.yml`.
   - Pulls updated container images from Docker Hub.
   - Performs zero-downtime rolling restart (`docker compose up -d --remove-orphans`).
   - Prunes stale Docker images to conserve VM disk storage.

### Required GitHub Repository Secrets

Navigate to **GitHub Repository -> Settings -> Secrets and variables -> Actions** and configure:

| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `DOCKERHUB_USERNAME` | Docker Hub user or organization | `someshtarra` |
| `DOCKERHUB_TOKEN` | Docker Hub Personal Access Token | `dckr_pat_xxxx` |
| `VM_HOST` | Public IP or FQDN of the Ubuntu VM | `54.210.xx.xx` |
| `VM_USERNAME` | SSH username on the VM | `ubuntu` |
| `VM_SSH_KEY` | Private SSH Key for authentication | `-----BEGIN OPENSSH PRIVATE KEY...` |
| `VM_PORT` | SSH Port (optional, defaults to 22) | `22` |

---

## 🌐 Nginx Reverse Proxy Setup Details

The application utilizes Nginx directly inside the `frontend` container to serve static assets and act as a reverse proxy:

```nginx
server {
    listen 80;
    server_name localhost;

    # Angular SPA static routing
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    # Nginx Reverse Proxy for Backend API
    location /api/ {
        proxy_pass http://mongodb:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # Inject CORS headers at proxy level
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Authorization' always;
    }
}
```

### Why this design?
1. **Zero Redundant Containers**: Merging the web server and reverse proxy into the frontend container avoids an extra proxy hop and eliminates an unnecessary container.
2. **Single Entry Point**: All browser requests originate from `http://<VM_IP>/`, routing `/` to the Angular SPA and `/api/` to Express.
3. **No Code Changes**: Container network sharing connects the backend to MongoDB on `localhost:27017`, and Nginx injects CORS headers at the proxy layer.

---

## 📸 Deliverables & Verification Screenshots

Place deliverables in [`docs/screenshots/`](docs/screenshots/):

### 1. CI/CD Configuration and Execution
![CI/CD Pipeline](docs/screenshots/01-cicd-workflow.png)
*Displays successful GitHub Actions run executing `build-and-push` and `deploy` jobs.*

### 2. Docker Image Build & Push Process
![Docker Hub Images](docs/screenshots/02-dockerhub-repositories.png)
*Displays Docker Hub repositories with pushed tags for `mean-backend` and `mean-frontend`.*

### 3. Cloud VM Deployment
![VM Deployment Status](docs/screenshots/03-vm-docker-compose-ps.png)
*Displays `docker compose ps` command output on Ubuntu VM confirming all containers are healthy.*

### 4. Working Application UI
![Working UI](docs/screenshots/04-application-ui.png)
*Displays the Angular CRUD UI accessed via `http://<VM_IP>/` demonstrating tutorial creation, retrieval, and search.*

### 5. Nginx Reverse Proxy Routing Verification
![Nginx Proxy Test](docs/screenshots/05-nginx-routing-test.png)
*Displays terminal `curl` tests confirming both `/` and `/api/tutorials` respond with HTTP 200 via Port 80.*

---

## 🧪 API Endpoints Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/tutorials` | Retrieve all tutorials (supports `?title=[keyword]` search) |
| `GET` | `/api/tutorials/:id` | Retrieve tutorial details by ID |
| `POST` | `/api/tutorials` | Create a new tutorial (`{ title, description, published }`) |
| `PUT` | `/api/tutorials/:id` | Update an existing tutorial |
| `DELETE` | `/api/tutorials/:id` | Delete a tutorial by ID |
| `DELETE` | `/api/tutorials` | Delete all tutorials |
| `GET` | `/api/tutorials/published` | Retrieve all published tutorials |
