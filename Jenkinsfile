pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
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

                    echo === NETWORK TEST ===
                    nslookup registry.npmjs.org || true
                    ping -c 3 registry.npmjs.org || true
                    curl -I https://registry.npmjs.org || true

                    echo "=== Installing Dependencies ==="
                    npm ci --verbose

                    echo "=== Building ==="
                    npm run build     
                '''
            }
        }
    }
}
