pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version

                    npm ci
                    npm run build

                    ls -la
                '''
            }
        }

        stage('test'){
            steps {
                sh '''
                    if [ -f "build/index.html" ]; then
                        echo "Index File was found first test pass"

                        npm test
                    else 
                        echo "Index was not found, failing tests..."
                        exit 1
                    fi
                '''
            }
        }
    }
}
