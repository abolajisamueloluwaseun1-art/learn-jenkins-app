pipeline {
    agent any

    triggers {
        githubPush()
    }
    
    stages {

        stage('Build')  {
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
                npm ci --cache /tmp/.npm-cache
                npm run build
                ls -la
                '''
            }
        }

        stage('Test') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                test -f build/index.html
                CI=true npm test
                '''
            }
        }

        stage('Deploy to render') {

            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }

            steps {

                withCredentials([
                    string(
                        credentialsId: 'NETLIFY_HOOK',
                        variable: 'NETLIFY_HOOK'
                    )
                ]) {

                    sh '''
                    curl -X POST -d {} https://api.netlify.com/build_hooks/6a0c37182ad5beabe4d9db75
                    '''
                }
            }
        }
    }

    post {

        always {
            junit 'test-results/junit.xml'
        }

        success {
            echo 'Pipeline completed - app deployed to Render!'
        }

        failure {
            echo 'Pipeline failed - deployment skipped.'
        }
    }
}