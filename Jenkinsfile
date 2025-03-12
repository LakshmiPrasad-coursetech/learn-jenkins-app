pipeline {
agent any

environment{
    /*NETLIFY_SITE_ID = 'ec48f01a-f465-4e09-99f5-1fd2ebf4f36c'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    */
    REACT_APP_VERSION = "1.0.$BUILD_ID"
    AWS_DEFAULT_REGION = 'us-west-1'
}

stages {

    stage('Deploy to AWS'){
        agent{
            docker{
                image 'amazon/aws-cli'
                reuseNode true
                args "--entrypoint=''"
            }
        }
        environment{
            AWS_S3_BUCKET = 'learn-jenkins-032025'
        }
        steps{
            withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
            aws --version
            aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
            '''
            }
            // aws s3 sync build s3://$AWS_S3_BUCKET
            
        }
    }

    stage('Build') {
        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
        steps {
            sh '''
            echo "Checking polling"
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
                        junit 'jest-results/junit.xml'
                    }
                }
            }

            stage('E2E') {
                agent {
                    docker {
                        image 'my-playwright'
                        reuseNode true
                    }
                }

                steps {
                    sh '''
                        serve -s build &
                        sleep 10
                        npx playwright test  --reporter=html
                    '''
                }

                post {
                    always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            }
        }
    }
 /*
    stage('Deploy Staging') {
        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
        steps {
            sh '''
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json

            '''

        script{
            env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        }

        }
        
    }
    */
    stage(' Deploy Staging') {
                agent {
                    docker {
                        image 'my-playwright'
                        reuseNode true
                    }
                }
               environment{
                    CI_ENVIRONMENT_URL = 'Staging URL to be set yet'
                }

                steps {
                    sh '''
                   
                    netlify --version
                    echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                        npx playwright test  --reporter=html
                    '''
                }

                post {
                    always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
    }
    /*stage('Approval'){
        steps{
                timeout(time: 15, unit: 'MINUTES'){
            input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
        }
    }*/

    stage(' Deploy Prod') {
                agent {
                    docker {
                        image 'my-playwright'
                        reuseNode true
                    }
                }
                environment{
                    CI_ENVIRONMENT_URL = 'https://roaring-piroshki-212a0b.netlify.app'
                }

                steps {
                    sh '''
                    node --version
                   
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod
                        npx playwright test  --reporter=html
                    '''
                }

                post {
                    always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
    }
}
}

