pipeline {
    agent none

    environment {
        IMAGE_NAME = 'bbcsite'
        CONTAINER_NAME = 'bbc'
        DOCKERHUB_USERNAME = 'dockerharikrishnan'
        KUBE_FILE = 'service.yaml'
        SERVICE_NAME = 'bbcnews'
    }

    stages {

        stage('Checkout') {
            agent {
                label 'docker'
            }
            steps {
                git branch: 'main', url: 'https://github.com/harikrishnan-knr/Nodejs-01.git'
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'docker'
            }
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Stop Old Container') {
            agent {
                label 'docker'
            }
            steps {
                sh 'docker stop ${CONTAINER_NAME} || true'
                sh 'docker rm ${CONTAINER_NAME} || true'
            }
        }

        stage('Run Docker Container') {
            agent {
                label 'docker'
            }
            steps {
                sh 'docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}:latest'
                sh 'docker ps'
            }
        }

        stage('Push Docker Image') {
            agent {
                label 'docker'
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
                git branch: 'main', url: 'https://github.com/harikrishnan-knr/Nodejs-01.git'
            }
        }

        stage('Kubernetes Version Check') {
            agent {
                label 'kuber'
            }
            steps {
                sh '''
                    kubectl version
                    eksctl version
                '''
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
                sh '''kubectl delete -f ${KUBE_FILE} || true
                kubectl apply -f ${KUBE_FILE}'''
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
                kubectl describe svc ${SERVICE_NAME}'''
            }
        }
    }

post {
        success {
            mail to: 'harikrishnan.cse7@gmail.com',
                 subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "Build succeeded: ${env.BUILD_URL}"
        }

        failure {
            mail to: 'harikrishnan.cse7@gmail.com',
                 subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "Build failed: ${env.BUILD_URL}"
        }
    } 
}
