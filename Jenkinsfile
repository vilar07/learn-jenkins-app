pipeline {
    agent any

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
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    args '--network host -u root:root'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install serve
                   node_modules/.bin/serve -s build &
                   sleep 10
                   npx playwright test
                '''
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
