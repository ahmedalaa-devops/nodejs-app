pipeline {
    agent any

    stages {
        
        stage('Build Image') {
            steps {
                dir('simple-node-js-react-npm-app') {
                    sh 'docker build -t react-app:${BUILD_NUMBER} .'
                }
            }
        }
        stage('Run Container') {
            steps {
                sh 'docker rm -f react-app-verify || true'
                sh 'docker run -d --name react-app-verify -p 8090:80 react-app:${BUILD_NUMBER}'
            }
        }
        stage('Verify') {
            steps {
                sh 'sleep 3'
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
            sh 'docker rm -f react-app-verify || true'
        }
        success {
            echo "Pipeline succeeded - Image: react-app:${BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed - check logs above"
        }
    }
}
