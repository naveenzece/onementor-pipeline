pipeline {
    agent any

    environment {
        CI = 'true'
        NODE_ENV = 'production'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git clean -fd'
            }
        }

        stage('Install Dependencies') {
            parallel {
                stage('API Install') {
                    steps {
                        dir('api') {
                            sh 'npm install'
                        }
                    }
                }
                stage('UI Install') {
                    steps {
                        dir('ui') {
                            sh 'npm install'
                        }
                    }
                }
            }
        }

        stage('Build UI') {
            steps {
                dir('ui') {
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Install PM2 globally
                    sh 'npm install -g pm2 --unsafe-perm=true'

                    // Stop existing apps if running
                    sh 'pm2 delete onementor-api onementor-ui || true'

                    // Start or reload ecosystem
                    sh 'pm2 startOrReload ecosystem.config.js --update-env'

                    // Save PM2 process list
                    sh 'pm2 save'

                    // Enable PM2 startup on reboot
                    sh 'pm2 startup | tail -1 | sudo bash || true'
                }
            }
        }

        stage('Configure Nginx') {
            steps {
                script {
                    sh 'sudo cp nginx/onementor.conf /etc/nginx/sites-available/onementor.conf'
                    sh 'sudo ln -sf /etc/nginx/sites-available/onementor.conf /etc/nginx/sites-enabled/onementor.conf'
                    sh 'sudo nginx -t'
                    sh 'sudo systemctl reload nginx'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
            echo 'API is running'
            echo 'UI is running'
            echo 'Both are accessible via Nginx'
        }
        failure {
            echo '❌ Deployment failed. Check the logs above for details.'
        }
        always {
            echo 'Build completed.'
        }
    }
}
