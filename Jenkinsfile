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
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--network host'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Test Stage"
                    test -f build/index.html
                    npm test
                '''
            }
        }
    }
}
