pipeline {
    agent { label 'ub22-agent' }

    environment {
        REGISTRY = "maesh0004"
        IMAGE_NAME = "springboot-app"
        DEPLOYMENT_NAME = "springboot-app"
        NAMESPACE = "default"
    }

    stages {

        stage("Check Tools") {
            steps {
                sh """
                    docker --version
                    java -version
                    kubectl version --client
                    mvn -v
                """
            }
        }

        stage("Build JAR") {
            steps {
                dir('app') {
                    sh """
                        mvn clean package -DskipTests
                        ls -l target/*.jar
                    """
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                dir('app') {
                    sh """
                        docker build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                                     -t ${REGISTRY}/${IMAGE_NAME}:latest .

                        docker images ${REGISTRY}/${IMAGE_NAME}
                    """
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                        echo "Logging into Docker Hub"
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                        docker push ${REGISTRY}/${IMAGE_NAME}:latest
                        docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage("Create K8s Secret") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                        kubectl delete secret dockerhub-secret \
                            --ignore-not-found -n ${NAMESPACE}

                        kubectl create secret docker-registry dockerhub-secret \
                            --docker-username=\$DOCKER_USER \
                            --docker-password="\$DOCKER_PASS" \
                            --docker-server=https://index.docker.io/v1/ \
                            --dry-run=client -o yaml | kubectl apply -f - -n ${NAMESPACE}
                    """
                }
            }
        }

        stage("Update Image Tag") {
            steps {
                sh """
                    sed -i "s/IMAGE_TAG/${BUILD_NUMBER}/g" k8s/app-deployment.yaml
                """
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                sh """
                    kubectl apply -f k8s/ -n ${NAMESPACE}

                    kubectl rollout status deployment/${DEPLOYMENT_NAME} \
                        -n ${NAMESPACE} --timeout=300s
                """
            }
        }
    }

    post {
        always {
            deleteDir()
        }
    }
}