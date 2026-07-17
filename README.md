# DevOps Engineering Assignment: Real-Time Chat App

Welcome! In this assignment, you are tasked with fixing a broken staging environment for our Real-Time Chat web application. 

A junior developer recently attempted to containerize this application using Docker and NGINX, but the deployment is currently failing on multiple fronts. Your job is to debug their configuration files and get the application fully operational via Docker Compose.

## System Architecture

The application is built using two primary containers:
1. **Backend (`backend`)**: A Python-based FastAPI server operating on Port 8000. It handles persistent, real-time WebSocket connections on the `/ws` endpoint.
2. **Frontend Proxy (`nginx`)**: An NGINX container mapped to Port 80. It is responsible for serving the static files from the `frontend/` directory, while simultaneously intercepting and reverse-proxying all WebSocket upgrade requests down to the backend container.

### Directory Structure
```text
realtime-chat-app/
├── app/
│   ├── main.py              # FastAPI application server
│   └── requirements.txt     # Python dependencies
├── frontend/
│   └── index.html           # Simple, styled single-page HTML client
├── Dockerfile               # Instructions to build the Python backend image
├── docker-compose.yml       # Composes both NGINX and Python Backend services
└── nginx.conf               # Configuration for NGINX routing and WS proxy
```

## Your Mission

If you run `docker-compose up -d --build` right now, the containers will start, but the application will not work. You need to debug and fix the following three critical issues:

### 1. Fix the Docker Binding (Container Networking)
The FastAPI backend container is refusing external connections—even from the NGINX container! 
* **Hint:** Look at how the `uvicorn` command is binding its host in the `Dockerfile`. Inside a Docker container, binding to `localhost` or `127.0.0.1` makes the service unreachable to other containers on the Docker network.

### 2. Fix the Missing User Interface (Volume Mounts)
If you navigate to `http://localhost` right now, you will likely see the default "Welcome to NGINX" page instead of the chat application.
* **Hint:** Check `docker-compose.yml`. How is the `nginx` container supposed to get access to the static HTML files located in the local `frontend/` directory? 

### 3. Fix the WebSocket Tunnel (Reverse Proxy Configuration)
Once the UI is visible, the chat app will continuously say "Disconnected" because the WebSocket handshake is failing.
* **Hint #1:** In `nginx.conf`, the `proxy_pass` is attempting to route to `localhost:8000`. Does `localhost` mean the same thing inside the NGINX container as it does on your laptop? How do containers communicate with each other in a Compose network?
* **Hint #2:** NGINX requires explicit headers to convert standard HTTP traffic into a persistent WebSocket tunnel. Some of the required `Upgrade` headers appear to be missing or disabled.

## Deliverables

Submit your finalized, corrected codebase. We will evaluate your submission by executing:

```bash
docker-compose up -d --build
```

If everything is configured correctly, we should instantly see the UI and be able to open multiple browser tabs at `http://localhost` to chat back and forth in real-time. Good luck!


Project Overview

This project demonstrates the deployment of a containerized real-time chat application using Docker, Docker Compose, NGINX, GitHub Actions, and AWS EC2.

The objective was to debug an intentionally broken deployment environment by fixing infrastructure-related issues without modifying the application logic.

After resolving the deployment issues, the application was successfully deployed on an AWS EC2 instance and automated using a GitHub Actions CI/CD pipeline.

Architecture
![alt text](Architecture.png)

Container Architecture

The application consists of two Docker containers:

- Backend Container
  - FastAPI application
  - Handles WebSocket connections
  - Runs on port 8000

- NGINX Container
  - Serves frontend static files
  - Reverse proxies WebSocket requests
  - Exposes port 80 to users

The containers communicate through the Docker Compose bridge network using service names.


Docker Networking

Docker Compose automatically creates a bridge network for all services.

NGINX communicates with the backend using the backend service name instead of localhost.

This allows inter-container communication without exposing internal services publicly.

Issues Identified and Fixes

Issue 1 – Backend Container Not Reachable

Problem:
The FastAPI server was bound to 127.0.0.1, preventing connections from other Docker containers.

Fix:
Updated the Dockerfile to bind Uvicorn to 0.0.0.0.

---

Issue 2 – Frontend Not Loading

Problem:
The frontend volume mount was commented out in docker-compose.yml, causing NGINX to serve its default welcome page.

Fix:
Enabled the frontend volume mount so NGINX could serve the application.

---

Issue 3 – WebSocket Connection Failed

Problem:
NGINX attempted to proxy WebSocket requests to localhost instead of the backend container.

Additionally, the required Upgrade and Connection headers were disabled.

Fix:
Updated proxy_pass to use the backend service name and enabled the required WebSocket upgrade headers.


CI/CD Pipeline

GitHub Actions has been configured to automatically deploy the application to AWS EC2.

Deployment workflow:

1. Push changes to the main branch.
2. GitHub Actions connects to the EC2 instance over SSH.
3. Pulls the latest repository changes.
4. Rebuilds Docker containers.
5. Restarts the application.
6. Removes unused Docker images.

This provides automated deployment with minimal manual intervention.

AWS Deployment

The application is deployed on an AWS EC2 Ubuntu instance.

Services are managed using Docker Compose.

The application is publicly accessible via the EC2 public IP.

Live Deployment

- **GitHub Repository:** https://github.com/kalyanteja18/devops
- **Application URL:** http://3.110.208.218/

The application has been successfully deployed on an AWS EC2 Ubuntu instance using Docker Compose. GitHub Actions is configured to automatically deploy the latest changes to the EC2 instance whenever code is pushed to the `main` branch.


Deployment Steps

1. Clone the repository

```bash
git clone https://github.com/kalyanteja18/devops.git
cd devops
```

2. Build and start the containers

```bash
docker compose up -d --build
```

3. Verify the running containers

```bash
docker ps
```

4. Access the application

Open the following URL in your browser:

Live Application: http://3.110.208.218/

Verification

The following functionality has been verified:

- Docker containers start successfully
- Frontend loads correctly
- NGINX serves static files
- WebSocket connections work successfully
- Multiple browser tabs can exchange messages in real time
- GitHub Actions automatically deploys changes to AWS EC2

Technologies Used

- Docker
- Docker Compose
- FastAPI
- NGINX
- GitHub Actions
- AWS EC2
- WebSockets


