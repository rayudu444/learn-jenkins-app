pipeline {
    agent any

    stages {

        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    rm -rf node_modules
                    npm ci
                    npm run build
                    test -f build/index.html
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                }
            }
            steps {
                sh 'npm test'
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.58.0-noble'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    npm install -g serve
                    serve -s build -l 3000 &
                    SERVER_PID=$!
                    sleep 5
                    npx playwright test
                    kill $SERVER_PID
                '''
            }
        }
    }

    post {
        always {
            script {
                if (fileExists('jest-results/junit.xml')) {
                    junit 'jest-results/junit.xml'
                } else {
                    echo 'JUnit report not found, skipping'
                }
            }
        }
    }
}
