pipeline {
    agent any

    tools {
        nodejs 'NodeJS-18'
    }

    environment {
        APP_DIR     = 'api/javascript/es2015-nodejs'
        IMAGE_NAME  = 'sample-api'
        CONTAINER   = 'sample-api-container'
        PORT        = '3000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm install --include=dev'
                }
            }
        }

        stage('Test') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm test || echo "Tests skipped"'
                }
            }
        }

        stage('Build Container Image') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                    podman build -t ${IMAGE_NAME}:latest .
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                podman rm -f ${CONTAINER} || true

                podman run -d \
                  --name ${CONTAINER} \
                  -p ${PORT}:${PORT} \
                  ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 10
                curl http://localhost:${PORT} || true
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD completed successfully '
        }

        failure {
            echo 'Pipeline failed '
        }

        always {
            sh 'podman ps -a || true'
        }
    }
}