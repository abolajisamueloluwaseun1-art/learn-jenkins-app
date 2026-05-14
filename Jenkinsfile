pipeline {
    agent any

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
                        credentialsId: 'RENDER_API_KEY',
                        variable: 'RENDER_API_KEY'
                    )
                ]) {

                    sh '''
                    curl -fsSL https://raw.githubusercontent.com/render-oss/cli/refs/heads/main/bin/install.sh | sh
                    render --version
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