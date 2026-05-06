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
                dir('simple-node-js-react-npm-app') {
                    sh '''
                        node -v || true
                        npm install
                        npx vitest run
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} simple-node-js-react-npm-app"
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
                        exit 1
                    fi

                    echo "PASSED"
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
            echo "✅ Pipeline succeeded"
        }

        failure {
            echo "❌ Pipeline failed"
        }
    }
}
