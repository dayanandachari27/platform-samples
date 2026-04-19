pipeline {
    agent any

    tools {
        nodejs 'NodeJS-18'
    }


    environment {
        APP_DIR = 'api/javascript/es2015-nodejs'
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/dayanandachari27/platform-samples'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm install --include=dev'
                }
            }
        }

        stage('Build') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm run build || echo "No build step"'
                }
            }
        }

        stage('Test') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm test || echo "No tests"'
                }
            }
        }

        stage('Run App (Optional)') {
            steps {
                dir("${APP_DIR}") {
                    sh 'nohup npm start &'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfullys'
        }
        failure {
            echo 'Pipeline failed '
        }
    }
}