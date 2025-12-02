pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '--network host'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
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
