# 🚀 CI/CD Pipeline using Jenkins, Docker & DockerHub

## 📌 Overview

This project implements an automated CI/CD pipeline using **Jenkins**, **Docker**, and **DockerHub**.

On every Git push:

* Jenkins is triggered via GitHub webhook
* Docker image is built and tagged with Jenkins `BUILD_NUMBER`
* Image is pushed to DockerHub
* Running container is stopped and redeployed automatically

The entire deployment process is fully automated without manual intervention.

---

## 🏗 Architecture

```
Developer
   │
   │ git push
   ▼
GitHub Repository
   │
   │ Webhook Trigger
   ▼
Jenkins Server (EC2 - Ubuntu)
   │
   │ Build Docker Image
   │ Tag with BUILD_NUMBER + latest
   ▼
DockerHub Registry
   │
   │ Pull latest image
   ▼
Docker Container (Running on same EC2)
   │
   ▼
Live Application (Port 80)


```


---

## ⚙️ Tech Stack

* Jenkins (Declarative Pipeline)
* Docker
* DockerHub (Image Registry)
* GitHub Webhooks
* Ubuntu EC2 (Jenkins Host)

---

## 🔁 How It Works

### 1️⃣ Code Push

Developer updates `index.html` and pushes to GitHub.

```
git add .
git commit -m "update version"
git push
```

---

### 2️⃣ Webhook Trigger

GitHub webhook sends a POST request to:

```
http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
```

Jenkins automatically starts the pipeline.

---

### 3️⃣ Build Stage

Jenkins:

* Builds Docker image
* Tags image as:

```
<dockerhub-username>/cicd-node-app:<BUILD_NUMBER>
<dockerhub-username>/cicd-node-app:latest
```

This ensures traceability of each build.

---

### 4️⃣ Push Stage

Pipeline logs into DockerHub using stored credentials and pushes:

* Versioned tag
* Latest tag

---

### 5️⃣ Deploy Stage

Pipeline executes:

* Pull latest image
* Stop existing container
* Remove container
* Run new container on port 80

Application becomes live immediately.

---

## 🧱 Jenkins Pipeline Stages

* Build Docker Image
* Login to DockerHub
* Push Versioned Image
* Deploy Container
* Post Success / Failure Handling

---

## 🔐 Security & Operational Controls

* DockerHub credentials stored securely in Jenkins Credentials Manager
* No hardcoded secrets in Jenkinsfile
* Webhook-based automation (no manual builds)
* Concurrent builds disabled
* Old builds automatically discarded (log rotation enabled)

---

## 📂 Repository Structure

```
cicd-node-app/
│
├── index.html
├── Dockerfile
├── Jenkinsfile
└── README.md
```

---

## 🐳 Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

---

## 🔧 Jenkinsfile (Simplified Logic)

```groovy
pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "your-dockerhub-username/cicd-node-app"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}:${env.BUILD_NUMBER}")
                    docker.build("${DOCKERHUB_REPO}:latest")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${DOCKERHUB_REPO}:${env.BUILD_NUMBER}"
                sh "docker push ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker pull ${DOCKERHUB_REPO}:latest
                docker stop cicd-app || true
                docker rm cicd-app || true
                docker run -d -p 80:80 --name cicd-app ${DOCKERHUB_REPO}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
```

---

## 🧪 Validation

After pushing code:

1. Jenkins auto-triggers pipeline
2. New image appears in DockerHub with incremented BUILD_NUMBER
3. Browser reflects updated content immediately

Example output in browser:

```
Deployed via Jenkins + Docker + Nginx
```

---

## 🧠 Why This Architecture?

### Why Jenkins?

* Orchestrates CI/CD workflow
* Automates build → push → deploy
* Manages credentials securely
* Enables webhook-based automation

---

### Why Docker?

* Provides consistent runtime environment
* Packages application with dependencies
* Enables versioned image tagging
* Simplifies deployment

---

### Why DockerHub?

* Central image registry
* Stores versioned artifacts
* Enables separation between build and runtime
* Supports rollback capability

---

## 🎯 What This Project Demonstrates

* CI/CD automation
* Webhook-based trigger mechanism
* Docker image versioning using BUILD_NUMBER
* Secure credential management in Jenkins
* Automated container redeployment

---

## 🚀 Outcome

This project simulates a production-style CI/CD workflow where application updates are automatically built, versioned, stored, and deployed without manual intervention.

