# 🚀 Blue-Green Node.js Deployment with Nginx + Docker Compose

This project demonstrates a **Blue/Green deployment** setup using **pre-built Node.js service images** behind an **Nginx reverse proxy**.  
Nginx automatically routes traffic, performs **health-based failover**, and supports **zero-downtime retries**.

---

## 🧩 Overview

### 🧩 Overview
🚀 Blue-Green Deployment with Nginx Reverse Proxy
🧭 Project Overview

This project demonstrates a Blue-Green Deployment strategy using Docker Compose and Nginx for zero-downtime failover between two Node.js applications — Blue and Green.

🧠 Real-life Scenario

In a production-like environment:

We deploy two identical Node.js containers: Blue and Green.

Nginx acts as a reverse proxy, routing traffic to the active app (Blue by default).

If Blue fails (timeout or 5xx error), Nginx automatically retries requests to Green within the same request — ensuring zero downtime.

Environment variables (.env) manage configuration, release IDs, and ports.

⚙️ 1. Project Structure

blue-green-deploy/
├── docker-compose.yml
├── nginx/
│   ├── nginx.conf.template
│   └── Dockerfile
├── .env
└── README.md

🧩 2. Environment Variables (.env)
# ---------- App Images ----------
BLUE_IMAGE=yimikaade/wonderful:devops-stage-two
GREEN_IMAGE=yimikaade/wonderful:devops-stage-two

# ---------- Release IDs ----------
RELEASE_ID_BLUE=v1.0-blue
RELEASE_ID_GREEN=v1.0-green

# ---------- Active Pool ----------
ACTIVE_POOL=blue  # or "green"

# ---------- Ports ----------
BLUE_PORT=8081
GREEN_PORT=8082
NGINX_PORT=8080

🖥️ 3. Commands Used in Real-Life Deployment (AWS EC2)Step 2: Update the Ubuntu Repository

Follow these exact steps when starting your AWS EC2 Ubuntu VM and deploying the project.

Step 1: Connect to the VM

From your local shell, SSH into your AWS Ubuntu instance:

ssh -i your-key.pem ubuntu@<your-ec2-public-ip>

Step 2: Update the Ubuntu Repository
sudo apt update -y && sudo apt upgrade -y

Step 3: Install Docker and Docker Compose
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo apt install docker-compose -y

Step 4: Install Git
sudo apt install git -y

tep 5: Clone Your GitHub Repository
After pushing your project from your local machine to GitHub:

git clone https://github.com/<your-username>/blue-green-deploy.git
cd blue-green-deploy

🐳 4. Run the Application
Start the containers in detached mode:
sudo docker compose up -d

🔍 5. Verify Deployment
Check Nginx health endpoint:
curl http://localhost:8080/healthz

Verify Blue app is active (default):
curl -i http://localhost:8080/version

💥 6. Test Failover to Green
Simulate a Blue app failure:
curl -X POST http://localhost:8081/chaos/start?mode=error

Retry the same request:
Expected headers:
X-App-Pool: green
X-Release-Id: v1.0-green

✅ No downtime — Nginx automatically reroutes traffic to Green.

# 🧱 7. Architecture
                       ┌──────────────────────────────┐
                       │        Client / User         │
                       │   (Browser, API Consumer)    │
                       └──────────────┬───────────────┘
                                      │
                                      ▼
                       ┌──────────────────────────────┐
                       │         AWS EC2 VM           │
                       │ (Ubuntu + Docker Installed)  │
                       └──────────────────────────────┘
                                      │
                                      ▼
             ┌────────────────────────────────────────────────┐
             │                Docker Network                   │
             │     (Created automatically by Docker Compose)   │
             └────────────────────────────────────────────────┘
               │                    │                    │
               ▼                    ▼                    ▼
     ┌────────────────┐   ┌────────────────┐   ┌────────────────────┐
     │   app_blue     │   │   app_green    │   │       nginx        │
     │  Node.js App   │   │  Node.js App   │   │ Reverse Proxy Layer│
     │ Port: 8081     │   │ Port: 8082     │   │ Exposed: 8080      │
     │ Health: /healthz│  │ Health: /healthz│  │                    │
     └────────────────┘   └────────────────┘   └────────────────────┘
            │                   │                     │
            └──────────┬────────┴──────────┬──────────┘
                       │                   │
                       ▼                   ▼
           Routes requests to Blue by default  
           On Blue failure → automatically retries to Green













Example .env file:
