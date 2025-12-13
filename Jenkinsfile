pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '83b9c8c3-d16f-4e82-8cb2-7ccea4cddea0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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
                    echo 'Small Change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
                echo 'Hello World'
            }
        }


        stage('Tests'){
            parallel {
                stage('Unit tests'){
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        echo 'Test stage'
                        sh '''
                        test -f build/index.html
                        echo $? 
                        npm test

                        '''
                    }
                        post {
                            always {
                                junit 'jest-results/junit.xml'
                                }
                        }
                }

                stage('E2E'){
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble' // same version as "@playwright/test": "^1.49.1",
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build & 
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }

                        post {
                            always {
                           
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }


    }


}