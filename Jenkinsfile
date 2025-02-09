pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '71a7ec1b-adc1-443e-81a5-8a4c9c4fa9a2'
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')

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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '-e NODE_ENV=test -e CI=true'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            test -f build/index.html
                            npm test -- --coverage
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'coverage', reportFiles: 'index.html', reportName: 'Coverage Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                stage('E2E') {
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
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            // environment {
            //     NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
            // }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify deploy --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --prod --dir=build
                '''
            }
        }
    }
}
