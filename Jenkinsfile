pipeline {
    agent any

    environment {
        BACKEND_IMAGE  = "aasamedchem-backend"
        FRONTEND_IMAGE = "aasamedchem-frontend"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        // Set your Docker Hub username here if you want to push images
        DOCKER_HUB_USER = "your-dockerhub-username"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git repository...'
                checkout scm
            }
        }

        stage('Install Dependencies & Test Build') {
            steps {
                echo 'Installing npm dependencies and verifying Vite build...'
                script {
                    if (isUnix()) {
                        sh 'npm install'
                        sh 'npm run build'
                    } else {
                        bat 'npm install'
                        bat 'npm run build'
                    }
                }
            }
        }

        stage('Docker Build Images') {
            steps {
                echo 'Building Docker images for Backend and Frontend...'
                script {
                    if (isUnix()) {
                        // Build Backend image
                        sh "docker build -f api/Dockerfile -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest ."
                        // Build Frontend image
                        sh "docker build -f Dockerfile -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest ."
                    } else {
                        // Build on Windows Jenkins agent
                        bat "docker build -f api/Dockerfile -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest ."
                        bat "docker build -f Dockerfile -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest ."
                    }
                }
            }
        }

        /* 
        // OPTIONAL: Uncomment this stage if you need to push images to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        if (isUnix()) {
                            sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                            sh "docker tag ${BACKEND_IMAGE}:latest ${DOCKER_HUB_USER}/${BACKEND_IMAGE}:latest"
                            sh "docker tag ${FRONTEND_IMAGE}:latest ${DOCKER_HUB_USER}/${FRONTEND_IMAGE}:latest"
                            sh "docker push ${DOCKER_HUB_USER}/${BACKEND_IMAGE}:latest"
                            sh "docker push ${DOCKER_HUB_USER}/${FRONTEND_IMAGE}:latest"
                        } else {
                            bat 'echo %DH_PASS% | docker login -u %DH_USER% --password-stdin'
                            bat "docker tag ${BACKEND_IMAGE}:latest ${DOCKER_HUB_USER}/${BACKEND_IMAGE}:latest"
                            bat "docker tag ${FRONTEND_IMAGE}:latest ${DOCKER_HUB_USER}/${FRONTEND_IMAGE}:latest"
                            bat "docker push ${DOCKER_HUB_USER}/${BACKEND_IMAGE}:latest"
                            bat "docker push ${DOCKER_HUB_USER}/${FRONTEND_IMAGE}:latest"
                        }
                    }
                }
            }
        }
        */

        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying application using Docker Compose...'
                script {
                    if (isUnix()) {
                        sh 'docker compose down || true'
                        sh 'docker compose up -d --build'
                    } else {
                        bat 'docker compose down'
                        bat 'docker compose up -d --build'
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build #${BUILD_NUMBER} completed and deployed successfully!"
        }
        failure {
            echo "Build #${BUILD_NUMBER} failed! Check console output for details."
        }
        always {
            echo 'Cleaning up dangling Docker images...'
            script {
                if (isUnix()) {
                    sh 'docker image prune -f || true'
                } else {
                    bat 'docker image prune -f'
                }
            }
        }
    }
}
