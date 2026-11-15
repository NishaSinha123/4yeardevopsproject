pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "aasamedchem-backend"
        FRONTEND_IMAGE  = "aasamedchem-frontend"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        // Securely injected from Jenkins Credentials (Secret text)
        DATABASE_URL    = credentials('DATABASE_URL')
        JWT_SECRET      = credentials('JWT_SECRET')
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

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube code quality scan...'
                script {
                    // Requires SonarQube server configured under:
                    // Manage Jenkins -> System -> SonarQube servers (name: 'SonarQube')
                    // Manage Jenkins -> Tools -> SonarQube Scanner (name: 'sonar-scanner')
                    try {
                        def scannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        withSonarQubeEnv('SonarQube') {
                            if (isUnix()) {
                                sh "${scannerHome}/bin/sonar-scanner"
                            } else {
                                bat "\"${scannerHome}\\bin\\sonar-scanner.bat\""
                            }
                        }
                    } catch (Exception e) {
                        echo "SonarQube tool step skipped or using fallback from PATH: ${e.message}"
                        withSonarQubeEnv('SonarQube') {
                            if (isUnix()) {
                                sh 'sonar-scanner'
                            } else {
                                bat 'sonar-scanner'
                            }
                        }
                    }
                }
            }
        }

        stage('Docker Build Images') {
            steps {
                echo 'Building Docker images for Backend and Frontend...'
                script {
                    if (isUnix()) {
                        sh "docker build -f api/Dockerfile -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest ."
                        sh "docker build -f Dockerfile -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest ."
                    } else {
                        bat "docker build -f api/Dockerfile -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest ."
                        bat "docker build -f Dockerfile -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest ."
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying application using Docker Compose with secure credentials...'
                script {
                    if (isUnix()) {
                        // Dynamically create .env from Jenkins credentials
                        sh """
                        cat <<EOF > .env
PORT=5000
DATABASE_URL=${DATABASE_URL}
JWT_SECRET=${JWT_SECRET}
NODE_ENV=production
EOF
                        """
                        sh 'docker rm -f aasamedchem-backend aasamedchem-frontend || true'
                        sh 'docker compose down || true'
                        sh 'docker compose up -d'
                    } else {
                        // On Windows Jenkins agent, write secure .env dynamically
                        bat """
@echo off
(
echo PORT=5000
echo DATABASE_URL=%DATABASE_URL%
echo JWT_SECRET=%JWT_SECRET%
echo NODE_ENV=production
) > .env
                        """
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
            echo "Build #${BUILD_NUMBER} completed, analyzed, and deployed successfully!"
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
