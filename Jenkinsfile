pipeline {
    agent { label 'windows-slave' }

    environment {
        PYTHON = 'python'
        VENV_DIR = '.venv'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gagankm27/Demo-Jenkins-pipeline.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Prepare Python venv') {
            steps {
                bat """
                if not exist %VENV_DIR% %PYTHON% -m venv %VENV_DIR%
                %VENV_DIR%\\Scripts\\python -m pip install --upgrade pip
                %VENV_DIR%\\Scripts\\pip install -r requirements.txt
                """
            }
        }

        stage('Compile / Build check') {
            steps {
                bat """
                %VENV_DIR%\\Scripts\\python -m compileall -q src
                """
            }
        }

        stage('Code Review - Static Analysis') {
            steps {
                bat """
                %VENV_DIR%\\Scripts\\pip install flake8 mypy
                %VENV_DIR%\\Scripts\\flake8 src tests
                %VENV_DIR%\\Scripts\\mypy src
                """
            }
        }

        stage('Unit Tests') {
            steps {
                bat """
                %VENV_DIR%\\Scripts\\pytest -q tests
                """
            }
        }

        stage('Metric Test - Coverage') {
            steps {
                bat """
                %VENV_DIR%\\Scripts\\pip install pytest-cov coverage
                %VENV_DIR%\\Scripts\\pytest --maxfail=1 --disable-warnings -q --cov=src --cov-report=term --cov-fail-under=80
                """
            }
        }

        stage('Package') {
            steps {
                bat """
                %VENV_DIR%\\Scripts\\pip install wheel setuptools
                %VENV_DIR%\\Scripts\\python setup.py sdist bdist_wheel
                """
            }
        }

        stage('Deploy') {
            steps {
                bat """
                if not exist C:\\deployments mkdir C:\\deployments
                xcopy dist\\* C:\\deployments\\ /Y /Q
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check console output.'
        }
        always {
            bat """
            if exist %VENV_DIR% rmdir /s /q %VENV_DIR%
            """
        }
    }
}
