
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Repository') {
            steps {
                echo 'AasaMedChem source code checked out successfully!'
            }
        }
    }
}
