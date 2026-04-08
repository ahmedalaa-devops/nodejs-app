pipeline {
    agent any

    environment {
        IMAGE_NAME = 'react-app'
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'react-app-verify'
        HOST_PORT = '8090'
    }

    stages {

        stage('Install & Test') {
            steps {
                dir('simple-node-js-react-npm-app') {
                    sh 'npm ci'
                    sh 'npx vitest run'
                }
            }
        }

        stage('Build Image') {
            steps {
                dir('simple-node-js-react-npm-app') {
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:80 \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    sleep 3
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:${HOST_PORT})
                    echo "HTTP Status: $STATUS"
                    if [ "$STATUS" != "200" ]; then
                        echo "Verification FAILED - got HTTP $STATUS"
                        exit 1
                    fi
                    echo "Verification PASSED"
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f ${CONTAINER_NAME} || true'
        }
        success {
            echo "Pipeline succeeded - Image: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed - check logs above"
        }
    }
}
