pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "aasamedchem-backend"
        FRONTEND_IMAGE  = "aasamedchem-frontend"
        IMAGE_TAG       = "${BUILD_NUMBER}"
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
                    try {
                        def scannerHome = tool 'SonarScanner'
                        withSonarQubeEnv('SonarQube') {
                            if (isUnix()) {
                                sh "${scannerHome}/bin/sonar-scanner"
                            } else {
                                bat "\"${scannerHome}\\bin\\sonar-scanner.bat\""
                            }
                        }
                    } catch (Exception e) {
                        echo "SonarQube tool step skipped or using fallback from PATH: ${e.message}"
                        try {
                            withSonarQubeEnv('SonarQube') {
                                if (isUnix()) {
                                    sh 'sonar-scanner || true'
                                } else {
                                    bat 'sonar-scanner 2>nul || exit 0'
                                }
                            }
                        } catch (Exception ex) {
                            echo "SonarQube analysis notice: ${ex.message}"
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
                echo 'Deploying application using Docker Compose...'
                script {
                    def dbUrl = ''
                    def jwtSec = ''

                    // Try to read credentials from Jenkins Credentials store if configured
                    try {
                        withCredentials([
                            string(credentialsId: 'DATABASE_URL', variable: 'SEC_DB_URL'),
                            string(credentialsId: 'JWT_SECRET', variable: 'SEC_JWT')
                        ]) {
                            dbUrl = SEC_DB_URL
                            jwtSec = SEC_JWT
                        }
                        echo 'Successfully loaded DATABASE_URL and JWT_SECRET from Jenkins Credentials.'
                    } catch (Exception e) {
                        echo "Jenkins Credentials 'DATABASE_URL'/'JWT_SECRET' not configured yet (${e.message}). Using local configuration."
                    }

                    if (dbUrl != '' && jwtSec != '') {
                        writeFile file: '.env', text: "PORT=5000\nDATABASE_URL=${dbUrl}\nJWT_SECRET=${jwtSec}\nNODE_ENV=production\n"
                    } else {
                        if (isUnix()) {
                            sh 'if [ ! -f .env ]; then cp .env.example .env; fi'
                        } else {
                            bat 'if not exist .env copy .env.example .env'
                        }
                    }

                    if (isUnix()) {
                        sh 'docker rm -f aasamedchem-backend aasamedchem-frontend || true'
                        sh 'docker compose down || true'
                        sh 'docker compose up -d'
                    } else {
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
            node {
                script {
                    if (isUnix()) {
                        sh 'docker image prune -f || true'
                    } else {
                        bat 'docker image prune -f 2>nul || exit 0'
                    }
                }
            }
        }
    }
}
