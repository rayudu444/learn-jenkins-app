pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '80c2dae7-025d-46d3-9826-16f4e97ef357'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        

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

        stage('AWS'){
            agent{
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '--entrypoint=""'
                }
            }

            environment{
                AWS_S3_BUCKET = 'jenkins-28112024'
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "Hello S3" > index.html
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
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
                        image 'my-playwright'
                        reuseNode true
                        // args '-u root:root'
                        }
                    }
                    steps{
                        sh '''
                        serve -s build & 
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
                image 'my-playwright'
                reuseNode true
                // args '-u root:root'
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'STAGE_URL_TO_BE_SET'
            }


            steps{
                sh '''
                netlify --version
                echo "Deploying to staging site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
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
                image 'my-playwright'
                reuseNode true
                // args '-u root:root'
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://heartfelt-granita-1c0fa4.netlify.app'
            }

            steps{
                sh '''
                netlify --version
                echo "Deploying to production site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build
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
