pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    echo "=== Info ==="
                    node --version
                    npm --version
                    free -m || true

                    echo "=== Installing Dependencies ==="
                    npm ci --verbose

                    echo "=== Building ==="
                    npm run build     
                '''
            }
        }
    }
}
