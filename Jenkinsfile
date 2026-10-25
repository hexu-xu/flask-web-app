pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    parameters {
        booleanParam(name: 'RUN_LOCAL_DEPLOY', defaultValue: false,
                     description: 'Start the built Flask app on this Windows machine after a successful build')
    }

    environment {
        PYTHON_EXE = 'C:\\Python314\\python.exe'
        VENV_PYTHON = '.venv\\Scripts\\python.exe'
    }

    stages {
        stage('Verify Python Path') {
            steps {
                bat '"%PYTHON_EXE%" --version'
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/hexu-xu/flask-web-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '"%PYTHON_EXE%" -m venv .venv'
                bat '"%VENV_PYTHON%" -m pip install -r requirements.txt pytest pytest-cov flake8 pyinstaller'
            }
        }

        stage('Lint') {
            steps {
                bat '"%VENV_PYTHON%" -m flake8 src tests'
            }
        }

        stage('Test') {
            steps {
                bat '"%VENV_PYTHON%" -m pytest tests -v --cov=src --cov-report=html'
            }
            post {
                success {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('Build') {
            steps {
                bat '"%VENV_PYTHON%" -m PyInstaller --onefile --name flask-web-app src\\app.py'
                archiveArtifacts artifacts: 'dist/flask-web-app.exe', fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                expression { return params.RUN_LOCAL_DEPLOY }
            }
            steps {
                withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                    powershell '''
                        $exe = Join-Path $env:WORKSPACE 'dist\\flask-web-app.exe'
                        Start-Process -FilePath $exe -WorkingDirectory $env:WORKSPACE -WindowStyle Hidden
                    '''
                }
            }
        }
    }
}
