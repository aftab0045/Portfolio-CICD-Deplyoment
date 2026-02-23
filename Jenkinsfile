pipeline {
    agent any

    environment {
        SERVER_IP = '3.91.244.2'
        SSH_CREDENTIAL = 'portfolio-key'
        REMOTE_USER = 'ubuntu'
        REMOTE_PATH = '/var/www/portfolio'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/aftab0045/Portfolio-CICD-Deplyoment.git'
            }
        }

        stage('Deploy to Target Server') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Reload Nginx') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            sudo systemctl reload nginx
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Portfolio deployed successfully!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}