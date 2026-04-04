pipeline {
    agent { label 'local'}
    environment {
        REGISTRY = "maesh0004"
        IMAGE_NAME = "springboot-app"
        DEPLOYMENT_NAME = "springboot-app"
        NAMESPACE = "default"
    }

    stages {

        stage('Start') {
            steps {
                echo 'Pipeline started'
            }
        }

        stage("Check Tools"){
            steps{
                bat 'docker --version'
                bat 'java -version'
                bat 'kubectl version --client'
                bat 'mvn -v'
            }
        }

        stage("Checkout Code"){
            steps{
                git branch: 'main', url: 'https://github.com/mahesheramalla/springboot-k8s-cicd.git'
            }
        }

        stage("Build JAR"){
            steps{
                dir('app') {
                    bat 'mvn clean package -DskipTests'
                    bat 'dir target\\*.jar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    bat """
                    docker build -t %REGISTRY%/%IMAGE_NAME%:%BUILD_NUMBER% -t %REGISTRY%/%IMAGE_NAME%:latest .
                    docker images %REGISTRY%/%IMAGE_NAME%
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo Logging into Docker Hub
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%

                    docker push %REGISTRY%/%IMAGE_NAME%:latest
                    docker push %REGISTRY%/%IMAGE_NAME%:%BUILD_NUMBER%
                    """
                }
            }
        }
        stage('Create K8s Secret') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    kubectl delete secret dockerhub-secret --ignore-not-found -n %NAMESPACE%

                    kubectl create secret docker-registry dockerhub-secret ^
                    --docker-username=%DOCKER_USER% ^
                    --docker-password=%DOCKER_PASS% ^
                    --dry-run=client -o yaml | kubectl apply -f - -n %NAMESPACE%
                    """
                }
            }
        }
        stage('Update Image Tag') {
            steps {
                bat """
                powershell -Command "(Get-Content k8s/app-deployment.yaml) `
                -replace 'IMAGE_TAG', '%BUILD_NUMBER%' |
                Set-Content k8s/app-deployment.yaml"
                """
            }
        }
        
        stage("Deploy to Kubernetes"){
            steps{
                bat """
                @echo off

                echo --- Check Kubernetes Client ---
                kubectl version --client

                echo --- Apply YAMLs ---
                kubectl apply -f k8s/ -n %NAMESPACE%

                echo --- Get Pods ---
                kubectl get pods -n %NAMESPACE%

                echo --- Wait for Rollout ---
                kubectl rollout status deployment/%DEPLOYMENT_NAME% -n %NAMESPACE% --timeout=300s
                if %errorlevel% neq 0 (
                    echo ERROR: Deployment rollout failed
                    kubectl describe deployment %DEPLOYMENT_NAME% -n %NAMESPACE%
                    kubectl get pods -n %NAMESPACE%
                    exit /b %errorlevel%
                )

                echo --- Deployment Successful ---
                kubectl get pods -n %NAMESPACE%
                kubectl get svc -n %NAMESPACE%
                kubectl get ingress -n %NAMESPACE%
                """
            }
        }
}
}