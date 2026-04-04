# Spring Boot Kubernetes CI/CD Pipeline

A complete CI/CD pipeline project demonstrating the deployment of a Spring Boot
application to Kubernetes using Jenkins, Docker, and Kubernetes manifests.

## Tech Stack

- **Application**: Java 17, Spring Boot 4.0.5
- **Build Tool**: Apache Maven
- **Containerization**: Docker
- **Orchestration**: Kubernetes (Minikube)
- **CI/CD**: Jenkins
- **Registry**: Docker Hub

## Prerequisites

- Java 17
- Maven 3.x
- Docker
- Kubernetes (Minikube)
- Jenkins (for CI/CD)
- kubectl

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

Application runs on http://localhost:8081

### 2. Docker Build

```bash
cd app
docker build -t springboot-app:latest .
docker run -p 8081:8081 springboot-app:latest
```

### 3. Kubernetes Deployment

```bash
minikube start
minikube addons enable ingress
kubectl apply -f k8s/
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8081:80
```

## CI/CD Pipeline

The Jenkins pipeline automates:

1. **Environment Check**: Verify tools (Docker, Java, kubectl, Maven)
2. **Code Checkout**: Pull latest changes
3. **Build**: Compile and package JAR
4. **Container Build**: Create Docker image
5. **Deploy**: Apply Kubernetes manifests

### Pipeline Stages

- **Build JAR**: `mvn clean package -DskipTests`
- **Build Docker Image**: `docker build -t springboot-app:latest .`
- **Deploy to K8s**: `kubectl apply -f k8s/`

## API Endpoints

- `GET /` - Returns hello message from CI/CD pipeline

## Configuration

### Application
```properties
spring.application.name=app
server.port=8081
```

### Kubernetes
- **Deployment**: 3 replicas, rolling updates
- **Service**: ClusterIP on port 8081
- **Ingress**: Routes / to service
