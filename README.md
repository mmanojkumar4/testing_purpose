

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
| Frontend          | HTML, CSS, JavaScript,Nginx |
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
├── backend/                # Flask backend service
│   ├── app/                # Application source code
│   │   ├── main.py         # API entry point
│   │   └── db.py           # Database connection logic
│   ├── tests/              # Backend unit tests
│   ├── requirements.txt   # Python dependencies
│   └── Dockerfile          # Backend Docker image build
│
├── frontend/               # Frontend service
│   ├── templates/          # HTML templates
│   ├── static/             # CSS, JS, static assets
│   └── Dockerfile          # Frontend Docker image build
│
├── scripts/                # Deployment automation
│   └── deploy-staging.sh   # Staging deployment script
│
├── docker-compose.yml      # Local multi-container setup
├── Jenkinsfile             # CI/CD pipeline definition
└── README.md               # Project documentation


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

###  CI Stagess
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

###  CD Stages – 

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


##  Non-Root User (Container Security)

Docker containers run as **root by default**, which is a security risk.
This project follows best practices by running the application as a **non-root user**.

**Implementation**

* Create a dedicated user (`appuser`)
* Assign ownership of application files
* Run the container as `appuser`

```dockerfile
RUN useradd -m appuser
WORKDIR /app
COPY app app
RUN chown -R appuser:appuser /app
USER appuser
```

**Benefits**

* Reduced attack surface
* Prevents system-level access
* Production-grade container security

---

##  Multi-Stage Docker Builds (Optimization)

Multi-stage builds separate **build-time** and **runtime** dependencies so only required artifacts are shipped.

**Why Used**

* Smaller image size
* Fewer dependencies
* Better security and faster deployments

**Implementation**

```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY app app
CMD ["python", "app/main.py"]
```

**Result**

* Image size reduced by ~50%
* No build tools in runtime image
* Cleaner, production-ready container

---

##  Database & Networking Flow

### Docker Compose Networking

Docker Compose creates a private internal network where all services can talk to each other using service names.

```
Frontend  --->  Backend  --->  PostgreSQL
   |            |               |
  Nginx        Flask          Volume
```

---

### Frontend → Backend

* Frontend runs on **Nginx**
* API calls are sent to the backend using the service name `backend`
* No `localhost` is used inside containers

Example:

```js
fetch("http://backend:5000/api/health")
```

---

### Backend → Database

* Backend connects to PostgreSQL using the service name `postgres_db`
* Database credentials are passed using environment variables

Example:

```env
DB_HOST=postgres_db
DB_PORT=5432
DB_NAME=appdb
```

---

### Database Persistence

* PostgreSQL uses a Docker volume
* Data is not lost when containers restart

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

---

### Key Points

* Containers communicate using **service names**
* Database runs only inside Docker network (secure)
* Data is persistent using volumes
* Same setup works for **local, staging, and CI/CD**

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
This section lists common issues encountered during local development, Docker execution, and CI/CD pipeline runs, along with practical solutions.
---

###  Containers Not Starting or Exiting Immediately

**Symptoms**

* `docker-compose up` exits
* Containers restart continuously or stop

**Resolution**

```bash
docker-compose ps
docker-compose logs
```

* Verify `CMD` or `ENTRYPOINT` in Dockerfiles
* Ensure correct working directory
* Check application crash logs

---

### Frontend Loads but Backend Is Not Reachable

**Symptoms**

* UI loads at port `9090`
* API calls fail or show “Backend not reachable”

**Resolution**

```bash
docker-compose logs backend
```

* Ensure backend listens on `0.0.0.0`
* Confirm frontend API URL points to backend service name
* Verify backend container is running

---

###  Database Connection Errors

**Symptoms**

* Backend crashes with database-related errors
* `/health` endpoint shows DB failure

**Resolution**

```bash
docker-compose logs postgres_db
```

* Ensure database service is running
* Verify environment variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`)
* Restart services:

```bash
docker-compose restart
```

---

###  Docker Compose Builds Successfully but Services Don’t Run

**Symptoms**

* `docker-compose build` succeeds
* Containers fail at runtime

**Resolution**

* Check exposed ports for conflicts
* Ensure correct file paths inside containers
* Inspect logs for missing dependencies

---

###  Jenkins Pipeline Fails During Docker Build

**Symptoms**

* Pipeline fails at `docker build`
* Snapshot or layer errors

**Resolution**

```bash
docker system prune -af
```

* Restart Docker service
* Ensure Jenkins has access to Docker socket:

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

---

###  GitHub Webhook Not Triggering Jenkins Build

**Symptoms**

* Code push does not start pipeline

**Resolution**

* Verify webhook URL ends with:

```
/github-webhook/
```

* Ensure Jenkins is publicly reachable (ngrok if running locally)
* Check GitHub → Webhooks → Recent Deliveries

---

###  Old Application Version Still Running After Deployment

**Symptoms**

* Latest code changes not visible

**Resolution**

```bash
docker-compose down
docker-compose pull
docker-compose up -d
```

* Remove unused images if required:

```bash
docker image prune -a
```

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














