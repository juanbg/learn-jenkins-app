pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c3261d33-838a-43b9-af57-a8e82d7134df'
        NETLIFY_AUTH_TOKEN = credentials('netlify')
        REACT_APP_VERSION = 'v1.0.0'
    }

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
                    node --version
                    npm --version

                    npm ci
                    npm run build

                    ls -la
                '''
            }
        }

        stage('Test'){
            parallel {
                stage ('Unit Testing') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
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

                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }                    
                stage ('E2E Testing') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10

                            npx playwright test --reporter=html
                        '''
                    }

                    //using the publisher plugin
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }                    
            }
        }
        
        stage('Deploy Stage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Simulating a NON PROD Deploy"
                    sleep 10

                '''
            }
        }

        stage('Prudction Rls Approval'){
            steps{
                timeout(time: 1, unit: 'MINUTES'){
                    input 'Ready to deploy in PRD?'
                }
            }
        }

        stage('Deploy PRD') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version

                    echo "Deploying in site_id = $NETLIFY_SITE_ID"

                    node_modules/.bin/netlify status

                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage ('E2E Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://timely-puppy-480967.netlify.app'
            }

            steps {
                sh '''
                    echo "=============================="
                    echo "*****ENV_VARIABLE_LOADED_LOCAL"
                    echo $CI_ENVIRONMENT_URL

                    #skipped 'cause it fails due netflify security layer
                    #npx playwright test --reporter=html

                    
                '''
            }

            //using the publisher plugin
            //post {
                //always {
                    //skipped 'cause it fails due netflify security layer
                    //publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                //}
            //}
        } 
    }
}
