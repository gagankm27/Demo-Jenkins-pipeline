pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                sh 'pip install -r requirements-dev.txt'
            }
        }

        stage('Lint') {
            steps {
                sh 'flake8 src'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest --cov=src'
            }
        }
    }
}
