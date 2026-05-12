pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Repo') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e
                            test -f README.md
                            echo "Repository validation passed"
                        '''
                    } else {
                        bat '''
                            if not exist README.md exit /b 1
                            echo Repository validation passed
                        '''
                    }
                }
            }
        }

        stage('Optional Python Checks') {
            when {
                expression { fileExists('pyproject.toml') || fileExists('requirements.txt') }
            }
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e
                            python3 --version || python --version
                            if [ -f pyproject.toml ]; then
                                python3 -m pip install -e . || python -m pip install -e .
                            elif [ -f requirements.txt ]; then
                                python3 -m pip install -r requirements.txt || python -m pip install -r requirements.txt
                            fi
                            if [ -d tests ]; then
                                python3 -m pytest -q || python -m pytest -q
                            fi
                        '''
                    } else {
                        bat '''
                            python --version
                            if exist pyproject.toml python -m pip install -e .
                            if exist requirements.txt python -m pip install -r requirements.txt
                            if exist tests python -m pytest -q
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}