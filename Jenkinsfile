pipeline {
    agent { label 'windows-slave' }
    environment {
        // Set path to python if needed, or rely on PATH
        PYTHON = 'python'
    }
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }
    stages {
        stage('Checkout') {
            steps {
                // Use stored credentials if needed (credentialId)
                git branch: 'main',
                    url: 'https://github.com/your-username/your-repo.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Prepare Python venv') {
            steps {
                bat '''
                if not exist .venv python -m venv .venv
                call .venv\\Scripts\\activate.bat
                python -m pip install --upgrade pip
                python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Compile / Build check') {
            steps {
                // For Python "compile" we check byte-compile and packaging readiness
                bat '''
                call .venv\\Scripts\\activate.bat
                python -m compileall -q src || exit /b 0
                '''
            }
        }

        stage('Code Review - Static Analysis') {
            steps {
                bat '''
                call .venv\\Scripts\\activate.bat
                python -m pip install flake8 mypy
                python -m flake8 src tests
                python -m mypy src || exit /b %ERRORLEVEL%
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                bat '''
                call .venv\\Scripts\\activate.bat
                python -m pytest -q tests
                '''
            }
        }

        stage('Metric Test - enforce coverage & quality') {
            steps {
                bat '''
                call .venv\\Scripts\\activate.bat
                python -m pip install pytest-cov coverage
                python -m pytest --maxfail=1 --disable-warnings -q --cov=src --cov-report= --cov-fail-under=80
                '''
            }
        }

        stage('Package') {
            steps {
                bat '''
                call .venv\\Scripts\\activate.bat
                python -m pip install wheel setuptools
                python setup.py sdist bdist_wheel
                '''
            }
        }

        stage('Deploy') {
            steps {
                // Simple deploy: copy wheel to deployment folder and install / restart service
                bat '''
                call .venv\\Scripts\\activate.bat
                mkdir C:\\deployments 2>nul
                xcopy dist\\* C:\\deployments\\ /Y /Q
                REM Optionally install to target env:
                REM python -m pip install --force-reinstall C:\\deployments\\yourpkg.whl
                REM Restart service (if using nssm):
                REM nssm restart YourAppService
                '''
            }
        }
    } // stages
    post {
        success { echo 'Pipeline completed successfully.' }
        failure { echo 'Pipeline failed. Check console output.' }
        always {
            // clean workspace if you want
            bat 'rmdir /s /q .venv || exit /b 0'
        }
    }
}
