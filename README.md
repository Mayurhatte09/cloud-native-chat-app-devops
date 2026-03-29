
---

# Cloud Native Chat App DevOps

A production-ready full-stack chat application built using modern DevOps practices.

<!--- I built a cloud-native chat application where I used Docker for containerization, Kubernetes for orchestration, and Jenkins for CI/CD to simulate a production DevOps environment.--->


---

## Table of Contents

- [Cloud Native Chat App DevOps](#cloud-native-chat-app-devops)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Features](#features)
  - [Tech Stack](#tech-stack)
  - [Architecture Overview](#architecture-overview)
  - [Project Structure](#project-structure)
  - [Prerequisites](#prerequisites)
  - [Setup `.env`](#setup-env)
  - [Step 1: Build Docker Images](#step-1-build-docker-images)
  - [Step 2: Apply Kubernetes Manifests](#step-2-apply-kubernetes-manifests)
  - [Step 3: Verify Deployments](#step-3-verify-deployments)
  - [Step 4: Configure Hosts File](#step-4-configure-hosts-file)
  - [Step 5: Access the App (Important for Ingress)](#step-5-access-the-app-important-for-ingress)
  - [CI/CD (Jenkins)](#cicd-jenkins)
    - [Pipeline Flow](#pipeline-flow)
  - [Step 6: Common Issues \& Fixes](#step-6-common-issues--fixes)
  - [Project Notes](#project-notes)
  - [👤 About the Architect](#-about-the-architect)

---

## Introduction
This is a production-ready cloud-native chat application demonstrating real-world DevOps practices including CI/CD, containerization, and Kubernetes orchestration.

---

## Features

- Real-time messaging using Socket.IO
- User authentication (JWT)
- Profile management
- Online/offline user status
- Dockerized frontend & backend
- Kubernetes deployment with Ingress
- MongoDB persistent storage
- CI/CD pipeline using Jenkins

---

## Tech Stack

- **Frontend:** React, TailwindCSS, Zustand
- **Backend:** Node.js, Express, Socket.IO
- **Database:** MongoDB
- **Containerization:** Docker
- **Orchestration:** Kubernetes
- **CI/CD:** Jenkins
- **Ingress:** NGINX

---

## Architecture Overview

```
User → Browser
       ↓
   Ingress (NGINX)
       ↓
 ┌───────────────┐
 │   Frontend    │
 │   React App   │
 └──────┬────────┘
        ↓ API Calls
 ┌───────────────┐
 │   Backend     │
 │ Node + Socket │
 └──────┬────────┘
        ↓
     MongoDB

CI/CD:
GitHub → Jenkins → Docker → Kubernetes
```

---

## Project Structure

```
full-stack_chatApp/
├── backend/            # Node.js backend
├── frontend/           # React frontend
├── kubernetes/         # Kubernetes manifests (deployments, services, ingress)
├── docker-compose.yml  # Optional local Docker Compose setup
├── Jenkinsfile
└── README.md

````

---

## Prerequisites

- Docker Desktop (Kubernetes enabled)
- kubectl
- Node.js >= 18 (optional)
- Git
- Browser

---

## Setup `.env`

```bash
cd backend
```

Create `.env`:

```
MONGODB_URI=mongodb://mongoadmin:secret@mongodb:27017/dbname?authSource=admin
JWT_SECRET=your_jwt_secret_key
PORT=5001
```

---

## Step 1: Build Docker Images

```bash
cd backend
docker build -t your-dockerhub/chatapp-backend:latest .

cd ../frontend
docker build -t your-dockerhub/chatapp-frontend:latest .
```

**Push images to Docker Hub:**

```bash
docker push mayrhatte09/chatapp-backend:latest
docker push mayrhatte09/chatapp-frontend:latest
```

---

## Step 2: Apply Kubernetes Manifests

1. Apply namespace:

```bash
kubectl apply -f kubernetes/namespace.yaml
```

2. Apply Persistent Volume & PVC for MongoDB

```bash
kubectl apply -f kubernetes/mongodb-pv.yaml
kubectl apply -f kubernetes/mongodb-pvc.yaml
```

1. Apply MongoDB deployment & service:

```bash
kubectl apply -f kubernetes/mongodb-deployment.yaml
kubectl apply -f kubernetes/mongodb-service.yaml
```

1. Apply backend deployment & service:

```bash
kubectl apply -f kubernetes/backend-deployment.yaml
kubectl apply -f kubernetes/backend-service.yaml
```

5. Apply frontend deployment & service:

```bash
kubectl apply -f kubernetes/frontend-deployment.yaml
kubectl apply -f kubernetes/frontend-service.yaml
```

6. Apply secrets (JWT secret):

```bash
kubectl apply -f kubernetes/secrets.yaml
```

1. Apply Ingress to route `chatapp.com`:

```
kubectl apply -f kubernetes/ingress.yaml
```

---

## Step 3: Verify Deployments

Check pods:

```bash
kubectl get pods -n kubernetes
```

Check services:

```bash
kubectl get svc -n kubernetes
```

Check ingress:

```bash
kubectl get ingress -n kubernetes
```

Ensure NGINX ingress controller is running:

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

(Optional) Port-forwarding:

```bash
kubectl port-forward service/backend -n kubernetes 5001:5001
kubectl port-forward service/frontend -n kubernetes 80:80
```

---

## Step 4: Configure Hosts File

Ingress uses a custom host (`chatapp.com`). You need to map it to localhost.

1. Open **Notepad as Administrator**:

```
Press Windows Key → type Notepad → Right Click → Run as administrator
```

1. Open the hosts file:

```
C:\Windows\System32\drivers\etc\hosts
```

1. Add the line:

```
127.0.0.1 chatapp.com
```

1. Save and flush DNS:

```bash
ipconfig /flushdns
```

1. Verify:

```bash
ping chatapp.com
# Should return 127.0.0.1
```

---

## Step 5: Access the App (Important for Ingress)

- Frontend: [http://chatapp.com/](http://chatapp.com/)
- Backend: [http://chatapp.com/api/](http://chatapp.com/api/)

---

## CI/CD (Jenkins)

### Pipeline Flow

1. Push code to GitHub
2. Jenkins pulls code
3. Builds Docker images
4. Pushes to Docker Hub
5. Deploys to Kubernetes

---

## Step 6: Common Issues & Fixes

| Issue                              | Fix                                                      |
| ---------------------------------- | -------------------------------------------------------- |
| chatapp.com not working            | Check hosts file and run `ipconfig /flushdns`            |
| Ingress no address                 | Install ingress controller                               |
| Backend not running                | Check pod logs                                           |
| Ingress shows `<none>` for ADDRESS | Ensure NGINX ingress controller is installed and running |
| Backend not working                | Verify backend service exposes port 5001 and pod is running                     |
| Frontend not loading               | Check frontend service and pod status                    |
| Secret not found                   | Apply `secrets.yaml` before backend deployment           |
| MongoDB connection error           | Verify `MONGODB_URI` and MongoDB pod status              |
| Pods stuck in Pending              | Check PVC/PV and cluster resources                       |
| ImagePullBackOff                   | Verify image name and push to Docker Hub                 |
| CrashLoopBackOff                   | Check logs and environment variables                     |
| Port not accessible                | Verify service ports and configuration                   |
| Ingress not routing                | Check ingress rules and service names                    |
| DNS not resolving                  | Verify hosts file entry                                  |
| Jenkins pipeline failed            | Check Jenkins console logs                               |
| Docker build failed                | Check Dockerfile and dependencies                        |
| Wrong namespace issue              | Use `-n kubernetes` in commands                          |
| Socket.IO not working              | Check backend port and WebSocket config                  |
| CORS error                         | Enable CORS in backend                                   |
| Backend API not reachable          | Verify backend service and endpoints                     |
| MongoDB pod restarting             | Check storage and logs                                   |
| Deployment not updating            | Reapply manifests or check image tag                     |
| High response time                 | Check pod resources and scaling                          |
| Service not found                  | Verify service name in configs                           |
| Env variables not working          | Check `.env` or Kubernetes secrets                       |
| Ingress 502 error                  | Backend service may be down                              |
| Permission denied (Docker/Jenkins) | Check Docker socket permissions                          |

---

## Project Notes

- MongoDB uses Persistent Volume and PVC to persist data.
- JWT secret stored in Kubernetes Secret and injected into backend.
- Frontend communicates with backend via `/api/` routed through Ingress.
- Docker Desktop Kubernetes is for local development; for production, use cloud K8s.


## 👤 About the Architect
<table align="center">
<tr>
<td align="center" width="160">
<img src="https://github.com/Mayurhatte09.png" width="120" style="border-radius: 10px; border: 3px solid #00D4FF;" />
</td>
<td>
<strong>Mayur Hatte</strong>



<em>DevOps & Cloud Infrastructure Engineer</em>




Focused on building self-healing, automated infrastructure. This playbook is a verified asset of <strong>MayurHatte09</strong>.
</td>
</tr>
</table>

<div align="center">
<sub>© 2026 | Mayur Hatte Design System</sub>

----
