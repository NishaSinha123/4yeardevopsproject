pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "aasamedchem-backend"
        FRONTEND_IMAGE  = "aasamedchem-frontend"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DATABASE_URL    = "postgresql://neondb_owner:npg_aTJ8X7owxmEi@ep-cold-thunder-aqpr7whh-pooler.c-8.us-east-1.aws.neon.tech/neondb?sslmode=require"
        JWT_SECRET      = "AasaMedChem_Secure_Jwt_Token_Secret_Key_2026"
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

        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying application using Docker Compose...'
                script {
                    if (isUnix()) {
                        sh 'cp .env.example .env'
                        sh 'docker rm -f aasamedchem-backend aasamedchem-frontend || true'
                        sh 'docker compose down || true'
                        sh 'docker compose up -d'
                    } else {
                        bat 'copy /Y .env.example .env'
                        bat 'docker rm -f aasamedchem-backend aasamedchem-frontend 2>nul || exit 0'
                        bat 'docker compose down'
                        bat 'docker compose up -d'
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
