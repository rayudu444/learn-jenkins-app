pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '80c2dae7-025d-46d3-9826-16f4e97ef357'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {

             agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            } 
            steps {
               sh '''
                echo 'small change'
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
               '''
            }
        }

        stage('Tests'){
            parallel{
                stage('Unit Tests'){
                    agent {
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    } 
                    steps{
                        sh '''
                            test -f build/index.html
                            npm run test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                stage('E2E'){
                    agent{
                        docker{
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                        // args '-u root:root'
                        }
                    }
                    steps{
                        sh '''
                        npm install  serve 
                        node_modules/.bin/serve -s build & 
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {

             agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            } 
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL= sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
           
        }

        stage('Staging E2E'){
            agent{
                docker{
                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                reuseNode true
                // args '-u root:root'
                }
            }

            environment{
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps{
                sh '''
                npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Production Approval') {
            steps {
                timeout(time:1, unit:'MINUTES') {
                    input message: '', ok: 'Yes, I am sure I want to deploy'
                }
            }
        }

        stage('Deploy Production') {

             agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            } 
            steps {
               sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to production site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build
               '''
            }
        }

        stage('Prod E2E'){
            agent{
                docker{
                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                reuseNode true
                // args '-u root:root'
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://heartfelt-granita-1c0fa4.netlify.app'
            }

            steps{
                sh '''
                npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
}
