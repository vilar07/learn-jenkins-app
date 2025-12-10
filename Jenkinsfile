pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '008cf74d-8452-44cb-bc1c-77e800165ff5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--network host'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small change"
                    ls -la
                    echo "=== Info ==="
                    node --version
                    npm --version

                    echo "=== Installing Dependencies ==="
                    npm ci

                    echo "=== Building ==="
                    npm run build     
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '--network host'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                            args '--network host'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    args '--network host'
                    reuseNode true
                }
            }

            steps {
                sh '''
                npm install netlify-cli@20.1.1 node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json   
                CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html
                npm test
                '''
            }
            post {
                always {
                    junit 'test-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Ready to deploy in production?', ok: 'Are you sure you want to deploy into production?'
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    args '--network host'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://genuine-licorice-cbd8a7.netlify.app'
            }

            steps {
                sh '''
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                sleep 10
                npx playwright test --reporter=html
                npm test
                '''
            }
            post {
                always {
                    junit 'test-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
