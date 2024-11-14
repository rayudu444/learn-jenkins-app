pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '80c2dae7-025d-46d3-9826-16f4e97ef357'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('Docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }

        stage('Build') {

             agent {
                docker{
                    image 'node:18'
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

        stage('Deploy Staging'){
            agent{
                docker{
                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                reuseNode true
                // args '-u root:root'
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'STAGE_URL_TO_BE_SET'
            }


            steps{
                sh '''
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to staging site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
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


        stage('Deploy Production'){
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
                npm --version
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to production site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build
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
