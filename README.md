# Spring Boot Kubernetes CI/CD Pipeline

A completed CI/CD pipeline project showing a Spring Boot application built with Maven, containerized with Docker, deployed to Kubernetes using kubeadm, and automated through Jenkins CI/CD.

## Tech Stack

- **Application**: Java 17, Spring Boot 4.0.5
- **Build Tool**: Apache Maven
- **Containerization**: Docker
- **Orchestration**: Kubernetes (`kubeadm`)
- **CI/CD**: Jenkins
- **Registry**: Docker Hub

## What this project demonstrates

- Completed end-to-end CI/CD automation with Jenkins
- Docker image build and registry push
- Kubernetes deployment with rolling updates
- Ingress-based external access
- Git-based SCM integration for Jenkins
- Real-world container orchestration with kubeadm

## Prerequisites

- Java 17
- Maven 3.x
- Docker
- Kubernetes cluster created with `kubeadm`
- `kubectl`
- `kubelet`
- Jenkins
- Ingress controller installed in the cluster

## Project Structure

```
springboot-k8s-cicd/
├── app/                          # Spring Boot application
│   ├── src/main/java/com/mahesh/app/
│   │   ├── AppApplication.java
│   │   └── controller/HelloController.java
│   ├── src/main/resources/application.properties
│   ├── Dockerfile
│   └── pom.xml
├── k8s/                          # Kubernetes manifests
│   ├── app-deployment.yaml
│   ├── app-service.yaml
│   └── app-ingress.yaml
├── Jenkinsfile                   # Jenkins pipeline
└── README.md
```

## Quick Start

### 1. Local Development

```bash
cd app
mvn clean package
java -jar target/app-1.jar
```

The application runs on:

```text
http://localhost:8081
```

### 2. Docker Build

```bash
cd app
docker build -t springboot-app:latest .
docker run -p 8081:8081 springboot-app:latest
```

### 3. Kubernetes Deployment

```bash
# Initialize a kubeadm cluster
sudo kubeadm init

# Configure kubectl for your user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Ensure an ingress controller is installed before deploying the app
# Deploy application manifests
kubectl apply -f k8s/
```

Then access the application through the ingress external node IP and port:

```text
http://<NODE_IP>:<NODE_PORT>/
```

> Example:
>
> ```text
> http://192.168.0.107:31704/
> ```

## CI/CD Pipeline

This project uses Jenkins to automate a Git-based build and deployment workflow.

Key pipeline actions:

1. **Environment Check**: Verify Docker, Java, Maven, and kubectl
2. **SCM Checkout**: Pull source code from GitHub
3. **Build**: Compile and package the Spring Boot JAR
4. **Docker Build**: Create and tag the container image
5. **Deploy**: Apply Kubernetes manifests to the cluster

### Pipeline Stages

- **Build JAR**: `mvn clean package -DskipTests`
- **Build Docker Image**: `docker build -t springboot-app:latest .`
- **Deploy to K8s**: `kubectl apply -f k8s/`

## Kubernetes Architecture

- **Deployment**: 3 replicas with rolling update strategy
- **Service**: `ClusterIP` service exposing port `8081`
- **Ingress**: Routes external traffic to the service

This architecture demonstrates production-style container orchestration and traffic routing for a stateless Spring Boot app.

## API Endpoints

- `GET /` - Returns a hello message from the Spring Boot application

## Configuration

### Application
```properties
spring.application.name=app
server.port=8081
```

### Kubernetes
- **Deployment**: 3 replicas, rolling update
- **Service**: ClusterIP on port `8081`
- **Ingress**: Routes `/` to the service

