pipeline {
    agent any
    environment {
        NEXUS_HOSTED = "localhost:30082"
        NEXUS_GROUP  = "localhost:30083"
        IMAGE_NAME   = "react-app"
        NEXUS_IMAGE  = "localhost:30082/react-app"
    }
    stages {
        stage('Install & Test') {
            steps {
                sh '
                    docker run --rm \
                        -v $(pwd)/simple-node-js-react-npm-app:/app \
                        -w /app \
                        node:20-alpine \
                        sh -c "ls -l && npm install && npx vitest run"
                '
            }
        }
        }
        stage('Build Image') {
            steps {
                dir('simple-node-js-react-npm-app') {
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }
        stage('Push to Nexus Hosted') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh """
                        echo \$NEXUS_PASS | docker login ${NEXUS_HOSTED} \
                            -u \$NEXUS_USER --password-stdin
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${NEXUS_IMAGE}:${BUILD_NUMBER}
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${NEXUS_IMAGE}:latest
                        docker push ${NEXUS_IMAGE}:${BUILD_NUMBER}
                        docker push ${NEXUS_IMAGE}:latest
                    """
                }
            }
        }
        stage('Run Container') {
            steps {
                sh 'docker rm -f react-app-verify || true'
                sh "docker run -d --name react-app-verify -p 8090:80 ${NEXUS_GROUP}/react-app:${BUILD_NUMBER}"
            }
        }
        stage('Verify') {
            steps {
                sh 'sleep 5'
                sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8090)
                    echo "HTTP Status: $STATUS"
                    if [ "$STATUS" != "200" ]; then
                        echo "Verification FAILED"
                        exit 1
                    fi
                    echo "Verification PASSED"
                '''
            }
        }
    }
    post {
        always {
            sh "docker logout ${NEXUS_HOSTED} || true"
            sh 'docker rm -f react-app-verify || true'
        }
        success {
            echo "✅ Pipeline succeeded - Image: ${NEXUS_IMAGE}:${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
