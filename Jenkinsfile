

pipeline {
    agent any

    environment {
        BACKEND_IMAGE  = "aasamedchem-backend"
        FRONTEND_IMAGE = "aasamedchem-frontend"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        // 1. Checkout source code from GitHub
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        // 2. Install dependencies and build React application
        stage('Install Dependencies & Test Build') {
            steps {
                echo 'Installing dependencies and building application...'

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

        // 3. SonarQube code quality analysis
        stage('SonarQube Analysis') {
            steps {
                echo 'Starting SonarQube code analysis...'

                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarScanner') {
                        if (isUnix()) {
                            sh "\"${scannerHome}/bin/sonar-scanner\""
                        } else {
                            bat "\"${scannerHome}\\bin\\sonar-scanner.bat\""
                        }
                    }
                }
            }
        }

        // 4. Build Docker images
        stage('Docker Build Images') {
            steps {
                echo 'Building Backend and Frontend Docker images...'

                script {
                    if (isUnix()) {
                        sh """
                            docker build -f api/Dockerfile \
                            -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                            -t ${BACKEND_IMAGE}:latest .
                        """

                        sh """
                            docker build -f Dockerfile \
                            -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                            -t ${FRONTEND_IMAGE}:latest .
                        """
                    } else {
                        bat """
                            docker build -f api/Dockerfile -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest .
                        """

                        bat """
                            docker build -f Dockerfile -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest .
                        """
                    }
                }
            }
        }

        // 5. Deploy application using Docker Compose
        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying application using Docker Compose...'

                script {

                    // Load secrets from Jenkins Credentials
                    withCredentials([
                        string(
                            credentialsId: 'DATABASE_URL',
                            variable: 'SEC_DB_URL'
                        ),
                        string(
                            credentialsId: 'JWT_SECRET',
                            variable: 'SEC_JWT'
                        )
                    ]) {

                        // Create .env file without exposing secrets in logs
                        writeFile(
                            file: '.env',
                            text: """PORT=5000
DATABASE_URL=${SEC_DB_URL}
JWT_SECRET=${SEC_JWT}
NODE_ENV=production
"""
                        )
                    }

                    echo 'Application environment configuration prepared.'

                    // Stop old containers and deploy updated application
                    if (isUnix()) {
                        sh 'docker compose down'
                        sh 'docker compose up -d'
                    } else {
                        bat 'docker compose down'
                        bat 'docker compose up -d'
                    }
                }
            }
        }
    }

    // Post-build actions
    post {

        success {
            echo "Build #${BUILD_NUMBER} completed successfully!"
            echo 'Application deployment completed.'
        }

        failure {
            echo "Build #${BUILD_NUMBER} failed!"
            echo 'Please check Jenkins Console Output for errors.'
        }

        always {
            echo 'Cleaning up dangling Docker images...'

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
