pipeline {
    agent any

    environment {
        // Set basic environment variables if needed
        CI = 'false'
    }

    stages {
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
                    // Create a production build of the Next.js application
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy') {
            steps {
                // Install PM2 globally if not present (optional, better pre-installed on agent)
                // sh 'npm install -g pm2'
                
                // Start or Reload applications using PM2
                sh 'pm2 startOrReload ecosystem.config.js'
                
                // Save the process list to be resurrected on reboot
                sh 'pm2 save'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
