pipeline {
    agent none

    environment {
        IMAGE_NAME = 'bbcnews'
        CONTAINER_NAME = 'bbcsite'
        DOCKERHUB_USERNAME = 'famidha'
        KUBE_FILE = 'service.yaml'
        SERVICE_NAME = 'bbcnews-ns'
    }

    stages {

        stage('Checkout') {
            agent {
                label 'dock'
            }
            steps {
                git branch: 'main', url: 'https://github.com/famidha2004/Nodejs-01.git'
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'dock'
            }
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Stop Old Container') {
            agent {
                label 'dock'
            }
            steps {
                sh 'docker stop ${CONTAINER_NAME} || true'
                sh 'docker rm ${CONTAINER_NAME} || true'
            }
        }

        stage('Run Docker Container') {
            agent {
                label 'dock'
            }
            steps {
                sh 'docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}:latest'
                sh 'docker ps'
            }
        }

        stage('Push Docker Image') {
            agent {
                label 'dock'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:latest ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest
                        docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Checkout for Kubernetes') {
            agent {
                label 'kuber'
            }
            steps {
                git branch: 'main', url: 'https://github.com/famidha2004/Nodejs-01.git'
            }
        }

        stage('Kubernetes Version Check') {
            agent {
                label 'kuber'
            }
            steps {
                sh 'kubectl version --client'
                sh 'eksctl version'
            }
        }

        stage('Verify Kubernetes Files') {
            agent {
                label 'kuber'
            }
            steps {
                sh 'ls -la'
            }
        }

        stage('Deploy to Kubernetes') {
            agent {
                label 'kuber'
            }
            steps {
                sh 'kubectl apply -f ${KUBE_FILE}'
            }
        }

        stage('Check Pods') {
            agent {
                label 'kuber'
            }
            steps {
                sh '''kubectl get pods -o wide
                kubectl get svc
                kubectl get deployments
                kubectl describe namespaces ${SERVICE_NAME}'''
            }
        }
    }
}
