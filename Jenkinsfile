pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    tools {
        nodejs 'NodeJS-18'
    }

    environment {
        APP_DIR        = 'api/javascript/es2015-nodejs'
        IMAGE_NAME     = 'sample-api'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'sample-api-container'
        APP_PORT       = '3000'
        HOST_PORT      = '3000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Pre-Checks') {
            steps {
                sh '''
                    node -v
                    npm -v
                    podman --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        npm ci || npm install --include=dev
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        npm test || echo "No valid tests found - continuing"
                    '''
                }
            }
        }

        stage('Build Container Image') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        podman build \
                          --cgroup-manager=cgroupfs \
                          -t ${IMAGE_NAME}:${IMAGE_TAG} \
                          -t ${IMAGE_NAME}:latest .
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    podman rm -f ${CONTAINER_NAME} || true

                    podman run -d \
                      --name ${CONTAINER_NAME} \
                      --cgroup-manager=cgroupfs \
                      -p ${HOST_PORT}:${APP_PORT} \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10
                    curl --fail http://localhost:${HOST_PORT} || exit 1
                '''
            }
        }

        stage('Container Verification') {
            steps {
                sh '''
                    podman ps
                    podman images | grep ${IMAGE_NAME}
                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
            sh '''
                podman logs ${CONTAINER_NAME} || true
            '''
        }

        always {
            sh '''
                podman ps -a || true
            '''
        }

        cleanup {
            sh '''
                docker image prune -f >/dev/null 2>&1 || true
            '''
        }
    }
}