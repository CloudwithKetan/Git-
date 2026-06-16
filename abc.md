# Flight Reservation Application Deployment Documentation

## 1. Introduction

This document describes the complete deployment process for the Flight Reservation Application. The application follows a microservices architecture and uses a modern DevOps toolchain for continuous integration and continuous deployment (CI/CD).

The deployment pipeline automates source code retrieval, application build, code quality analysis, containerization, image publishing, Kubernetes deployment, and frontend hosting.

---

# 2. Project Overview

The project consists of three major components:

### Backend Services

* Flight Reservation Service
* Flight Check-In Service

### Frontend

* React/Vite Application

### Database

* MySQL Database hosted on AWS RDS

---

# 3. Technology Stack

| Category           | Tool                 |
| ------------------ | -------------------- |
| Source Control     | GitHub               |
| CI/CD              | Jenkins              |
| Build Tool         | Maven                |
| Code Quality       | SonarQube            |
| Containerization   | Docker               |
| Container Registry | Docker Hub           |
| Orchestration      | Kubernetes (K3s/EKS) |
| Database           | AWS RDS MySQL        |
| Frontend Hosting   | AWS S3               |
| Cloud Platform     | AWS                  |

---

# 4. Architecture

```text
Developer
    |
    v
 GitHub Repository
    |
    v
 Jenkins Pipeline
    |
    +-------------------+
    |                   |
    v                   v
 SonarQube          Docker Build
    |                   |
    +--------+----------+
             |
             v
        Docker Hub
             |
             v
      Kubernetes Cluster
             |
    +--------+---------+
    |                  |
    v                  v
Reservation      Check-In Service
 Service
             |
             v
        AWS RDS MySQL

Frontend
    |
    v
AWS S3 Static Website
```

---

# 5. AWS Infrastructure Setup

## 5.1 Jenkins Server

Create an EC2 instance with the following configuration:

| Parameter     | Value        |
| ------------- | ------------ |
| OS            | Ubuntu 22.04 |
| Instance Type | t3.medium    |
| Storage       | 20 GB        |

### Security Group Rules

| Port | Purpose   |
| ---- | --------- |
| 22   | SSH       |
| 8080 | Jenkins   |
| 9000 | SonarQube |

---

## 5.2 Kubernetes Server

Create a second EC2 instance for Kubernetes.

| Parameter     | Value        |
| ------------- | ------------ |
| OS            | Ubuntu 22.04 |
| Instance Type | t3.medium    |

### Security Group Rules

| Port        | Purpose           |
| ----------- | ----------------- |
| 22          | SSH               |
| 80          | HTTP              |
| 443         | HTTPS             |
| 30000-32767 | NodePort Services |

---

## 5.3 RDS Database

Create an AWS RDS MySQL instance.

Configuration:

* Engine: MySQL 8.0
* Database Name: flightdb
* Public Access: No
* Security Group: Allow port 3306 from Kubernetes Security Group

---

## 5.4 Frontend Hosting

Create an S3 bucket.

Example:

```text
flight-frontend-bucket
```

Enable:

* Static Website Hosting
* Public Read Access (or CloudFront)

---

# 6. Jenkins Installation

Install Java:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Install Jenkins:

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Access Jenkins:

```text
http://<JENKINS-IP>:8080
```

---

# 7. Docker Installation

```bash
curl -fsSL https://get.docker.com | sh

sudo usermod -aG docker jenkins

sudo systemctl restart jenkins
```

Verify:

```bash
docker --version
```

---

# 8. Maven Installation

```bash
sudo apt install maven -y
```

Verify:

```bash
mvn -version
```

---

# 9. AWS CLI Installation

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o awscliv2.zip

unzip awscliv2.zip

sudo ./aws/install
```

Configure:

```bash
aws configure
```

Provide:

* AWS Access Key
* AWS Secret Key
* Region

---

# 10. SonarQube Installation

Run SonarQube using Docker:

```bash
docker run -d \
--name sonarqube \
-p 9000:9000 \
sonarqube:lts-community
```

Access:

```text
http://<JENKINS-IP>:9000
```

Create:

* SonarQube Project
* Sonar Token

---

# 11. Kubernetes Installation

Install K3s:

```bash
curl -sfL https://get.k3s.io | sh -
```

Verify:

```bash
kubectl get nodes
```

Expected output:

```text
Ready
```

---

# 12. Database Configuration

Update the backend configuration files.

Reservation Service:

```properties
spring.datasource.url=jdbc:mysql://<RDS-ENDPOINT>:3306/flightdb
spring.datasource.username=admin
spring.datasource.password=password
```

Check-In Service:

```properties
spring.datasource.url=jdbc:mysql://<RDS-ENDPOINT>:3306/flightdb
spring.datasource.username=admin
spring.datasource.password=password
```

Commit and push changes to GitHub.

---

# 13. Jenkins Configuration

Install Plugins:

* Pipeline
* Git
* Docker
* Docker Pipeline
* SonarQube Scanner
* Kubernetes
* Workspace Cleanup

---

# 14. Jenkins Credentials

Create the following credentials:

## SonarQube Token

Credential ID:

```text
sonar-cred
```

---

## Docker Hub

Credential ID:

```text
dockerhub-cred
```

---

## AWS Credentials

Credential ID:

```text
aws-cred
```

---

# 15. Pipeline Configuration

## Reservation Backend Pipeline

Create a Pipeline Job.

Name:

```text
reservation-backend
```

Repository:

GitHub Repository URL

Script Path:

```text
FlightReservationApplication/jenkinsfile.jdp
```

---

## Check-In Backend Pipeline

Name:

```text
checkin-backend
```

Script Path:

```text
FlightCheckInApplication/Jenkinsfile
```

---

## Frontend Pipeline

Name:

```text
frontend
```

Script Path:

```text
frontend/jenkins.groovy
```

---

# 16. Deployment Process

### Step 1

Trigger Reservation Backend Pipeline.

This stage performs:

* Git Checkout
* Maven Build
* SonarQube Analysis
* Docker Build
* Docker Push
* Kubernetes Deployment

Verify:

```bash
kubectl get pods
```

---

### Step 2

Trigger Check-In Backend Pipeline.

Verify:

```bash
kubectl get pods
```

---

### Step 3

Trigger Frontend Pipeline.

This stage:

* Installs Dependencies
* Builds React Application
* Uploads Static Files to S3

Verify website accessibility through the S3 endpoint.

---

# 17. Validation

Verify Kubernetes Resources:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

Verify Docker Images:

```bash
docker images
```

Verify Jenkins:

```bash
sudo systemctl status jenkins
```

Verify SonarQube:

```text
http://<JENKINS-IP>:9000
```

---

# 18. Expected Outcome

After successful deployment:

* Reservation Service running in Kubernetes
* Check-In Service running in Kubernetes
* Frontend hosted on AWS S3
* Database running on AWS RDS
* CI/CD fully automated through Jenkins

---

# 19. Conclusion

The Flight Reservation Application deployment implements a complete DevOps workflow using GitHub, Jenkins, SonarQube, Docker, Kubernetes, AWS RDS, and S3. The solution provides automated build, testing, code quality validation, container deployment, and frontend hosting, enabling reliable and repeatable software delivery.
