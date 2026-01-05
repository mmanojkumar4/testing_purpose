

#  CI/CD Capstone Project

## 1. Project Overview

This project demonstrates a **complete end-to-end CI/CD pipeline** implemented using **Docker, Docker Compose, Jenkins, and GitHub Webhooks** for a **2-tier containerized web application**.

The primary goal of this project is to **automate the full application lifecycle** — from source code commit to deployment — while following real-world DevOps best practices, including containerization, automated testing, security scanning, and staged deployments.

Every code push to GitHub automatically triggers the Jenkins pipeline, which builds, tests, scans, and deploys the application without manual intervention.

---

## 2. Problem Statement

**Build a complete CI/CD system that automatically tests, builds, and deploys a simple web application through staging environments using Docker and Jenkins.**

---

## 3. Technology Stack

| Layer             | Technology            |
| ----------------- | --------------------- |
| Frontend          | HTML, CSS, JavaScript |
| Backend           | Python Flask          |
| Database          | PostgreSQL            |
| CI/CD             | Jenkins               |
| Containerization  | Docker                |
| Orchestration     | Docker Compose        |
| Security Scanning | Trivy                 |
| Webhooks          | GitHub Webhook        |
| Version Control   | GitHub                |
| Webhook Exposure  | ngrok                 |

---

## 4. High-Level Architecture

### Architecture Flow

```
Developer
   |
   | Git Push
   v
GitHub Repository
   |
   | Webhook
   v
Jenkins Pipeline
   |
   +--> Build Images
   +--> Run Tests
   +--> Security Scan
   +--> Deploy to Staging
   |
   v
Docker Containers
   |
Frontend --> Backend --> PostgreSQL
```

 * Fully automated
 * Webhook-driven
 * No manual pipeline triggers

---

## 5. Application Architecture

### 5.1 Frontend

* Static HTML/CSS + JavaScript
* Served via **Nginx**
* Displays:

  * Backend connectivity status
  * Database connection status
  * Application health

### 5.2 Backend

* Flask REST API
* Exposes endpoints:

  * `/health` – Application & DB health
* Uses **environment variables** for database connectivity

### 5.3 Database

* PostgreSQL running in a container
* Persistent data storage using Docker volumes

---

## 6. Project Structure

```
ci-cd-flask-project/
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   └── db.py
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/
│   ├── templates/
│   ├── static/
│   └── Dockerfile
│
├── scripts/
│   └── deploy.sh
│
├── docker-compose.yml
├── Jenkinsfile
└── README.md
```

---

## 7. Dockerization Strategy

### Backend Dockerfile Best Practices

*  Multi-stage build
*  Reduced image size
*  Non-root user
*  Optimized layer caching

### Frontend Dockerfile Best Practices

* Based on `nginx:alpine`
* Minimal runtime footprint
* Static content only

### Image Optimization

| Image                   | Size      |
| ----------------------- | --------- |
| Backend (Multi-stage)   | ~55–60 MB |
| Frontend (Nginx Alpine) | ~20–25 MB |

---

## 8. Docker Compose Configuration

Docker Compose is used for:

* Local development
* Staging deployment
* Multi-container orchestration

### Services

* `frontend`
* `backend`
* `postgres`

### Features

* Custom Docker network
* Named volume for DB persistence
* Environment-based configuration

---

## 9. CI/CD Pipeline Using Jenkins

The entire CI/CD workflow is defined in a **Jenkinsfile**.

### Pipeline Stages

1. Checkout Code
2. Build Backend Docker Image
3. Run Unit Tests inside Container
4. Build Frontend Docker Image
5. Security Scan using Trivy
6. Deploy to Staging Environment


##  Jenkins – CI/CD & Webhook Flow

### Why Jenkins?

Jenkins is used as the **CI/CD automation tool** in this project.
It automatically builds, tests, and deploys the application whenever code is pushed to GitHub.

---

### Jenkins + GitHub Webhook Flow

```
Developer pushes code to GitHub
        ↓
GitHub Webhook triggers Jenkins
        ↓
Jenkins Pipeline starts automatically
        ↓
Build → Test → Deploy
```

* No manual build trigger
* Fully automated CI/CD

---

### What Jenkins Does in This Project

Jenkins performs the following steps:

1. **Checkout code** from GitHub
2. **Build Docker images** for backend and frontend
3. **Run tests inside Docker containers**
4. **Scan images using Trivy**
5. **Deploy application using Docker Compose**
6. **Verify backend health endpoint**

---

### Jenkins Pipeline 

```
Checkout Code
   ↓
Build Backend Image
   ↓
Run Backend Tests
   ↓
Build Frontend Image
   ↓
Security Scan (Trivy)
   ↓
Deploy to Staging
```

---

---

### Webhook Implementation

* GitHub Webhook is configured to call Jenkins
* Jenkins endpoint: `/github-webhook/`
* ngrok is used to expose Jenkins when running locally
* Every push triggers the pipeline automatically

* Continuous Integration achieved
* Continuous Deployment enabled

---


## 10. Continuous Integration (Webhook)

* GitHub Webhook triggers Jenkins automatically
* No manual “Build Now”
* Jenkins exposed using **ngrok** for webhook access

* True Continuous Integration
* Real-time automation

---

###  CI Flow Diagram (Logical)

```
Developer
   |
   | git push
   v
GitHub Repository
   |
   | Webhook Trigger
   v
Jenkins CI Pipeline
   |
   +--> Checkout Code
   |
   +--> Build Backend Image
   |
   +--> Run Unit Tests (Pytest)
   |
   +--> Build Frontend Image
   |
   +--> Trivy Security Scan
```

---

###  CI Stages – What We Did

#### 1️ Code Checkout

Pulls latest code automatically from GitHub.

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

* Ensures Jenkins always works with latest commit
* Triggered automatically via webhook

---

#### 2️ Backend Image Build

Builds Docker image for Flask backend.

```groovy
stage('Build Backend Image') {
    steps {
        bat 'docker build -t flask-backend backend'
    }
}
```

* Uses **multi-stage Dockerfile**
* Cached layers for faster builds

---

#### 3️ Unit Testing Inside Container

Runs tests inside Docker, not locally.

```groovy
stage('Run Backend Tests') {
    steps {
        bat 'docker build -t flask-backend-test --target test backend'
        bat 'docker run --rm flask-backend-test'
    }
}
```

 Real container testing
 Pipeline fails if tests fail
 No “works on my machine” issue

---

#### 4️ Frontend Image Build

Builds Nginx-based frontend image.

```groovy
stage('Build Frontend Image') {
    steps {
        bat 'docker build -t flask-frontend frontend'
    }
}
```

* Lightweight image
* Static content served via Nginx

---

#### 5️ Security Scan (Trivy)

Scans images for vulnerabilities.

```groovy
stage('Security Scan') {
    steps {
        bat '''
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image flask-backend
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image flask-frontend
        '''
    }
}
```

* Detects CVEs
* Industry-grade security practice
* Integrated directly into CI

---


## 11. Continuous Deployment

Deployment is fully automated for the **staging environment**.

### Deployment Actions

* Stop old containers
* Build latest images
* Start new containers
* Verify application health

Deployment is handled using **Docker Compose**.

---
##  Continuous Deployment (CD) Flow


```
CI Success
   |
   v
Staging Deployment Stage
   |
   +--> Stop Old Containers
   |
   +--> Pull / Build Latest Images
   |
   +--> Start New Containers
   |
   +--> Verify Application Health
```

---

###  CD Stage – What We Did

####  Deploy to Staging (Docker Compose)

```groovy
stage('Deploy to Staging') {
    steps {
        bat '''
        docker-compose down
        docker-compose up -d --build
        '''
    }
}
```

 Stops old containers
 Starts fresh containers
 Uses same images tested in CI
 Acts as **staging environment**

---



###  Webhook Trigger Flow

```
Developer Pushes Code
        |
        v
 GitHub Webhook
        |
        v
   Jenkins Endpoint
        |
        v
   CI/CD Pipeline Starts Automatically
```

###  Key Points

* No manual Jenkins build
* Every push = auto build
* Implemented using **GitHub Webhook + ngrok**



---

##  Multi-Stage Docker Build 

### Why Multi-Stage

* Smaller images
* Better security
* Faster deployments

### Backend Dockerfile Concept

```dockerfile
# Builder stage
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Runtime stage
FROM python:3.11-slim
RUN useradd -m appuser
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY app app
USER appuser
CMD ["python", "app/main.py"]
```

* Builder stage removed from final image
* Non-root user used
* Only required dependencies included

---

##  Database & Networking Flow

### Docker Compose Networking

```
Frontend  ---> Backend  ---> PostgreSQL
   |            |              |
  Nginx        Flask         Volume
```

---


## 12. Deployment Script (`deploy_staging.sh`)

```bash
#!/bin/bash
echo "Stopping old containers..."
docker-compose down

echo "Starting updated containers..."
docker-compose up -d --build

echo "Deployment completed successfully"
```

 Can be reused locally or in Jenkins
 Matches real-world deployment scripts

---

##  Verification After Deployment

### Backend Health Check

```bash
curl http://localhost:5000/health
```

Expected:

```json
{
  "status": "UP",
  "database": "CONNECTED"
}
```

### Frontend

```
http://localhost:9090
```

* Confirms full end-to-end flow


---

## 13. Runtime Health Verification

### Backend Health Check

```bash
curl http://localhost:5000/health
```

Expected Response:

```json
{
  "status": "UP",
  "database": "CONNECTED"
}
```

### Frontend

```
http://localhost:9090
```

---

## 14. Security Scanning with Trivy

* Integrated directly into Jenkins pipeline
* Scans Docker images for vulnerabilities
* Pipeline fails on **HIGH / CRITICAL** issues

* Security-first CI/CD approach

---

## 15. Environment Strategy

Although the same Docker Compose file is reused, environments are logically separated:

### Development

* Local execution
* Manual docker-compose

### Staging

* Jenkins-managed deployment
* Fully automated
* Pre-production validation

### Environment Variables Used

```yaml
environment:
  DB_HOST: postgres
  DB_PORT: 5432
  DB_NAME: testdb
  DB_USER: postgres
  DB_PASSWORD: postgres
```

* No hardcoded credentials
* Docker internal DNS used
* Persistent volume for DB

---

## 16. Troubleshooting Guide

### Backend Not Reachable

```bash
docker-compose logs backend
```

### Database Connection Issues

```bash
docker-compose logs postgres
```

### Webhook Not Triggering

* Verify webhook URL ends with `/github-webhook/`
* Ensure Jenkins is reachable (ngrok)

### Trivy Scan Fails

* Update base images
* Fix vulnerable dependencies

---

## 17. What This Project Demonstrates

*  End-to-end CI/CD automation
*  Webhook-driven Jenkins pipeline
*  Containerized testing
*  Multi-stage Docker builds
*  Security scanning integration
*  Real staging deployment flow
*  Industry-aligned DevOps practices

---

## 18. Conclusion

This capstone project successfully implements a **real-world CI/CD pipeline** using Docker and Jenkins. It automates the complete workflow from code commit to deployment while ensuring security, consistency, and reliability.

The integration of **GitHub Webhooks, Jenkins pipelines, Docker multi-stage builds, security scanning, and automated deployments** demonstrates strong practical DevOps knowledge suitable for real production environments and technical interviews.

---






















