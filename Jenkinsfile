pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install --legacy-peer-deps'
            }
        }

        stage('Build Angular App') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['aws-ec2-ssh-key']) {
                    bat "ssh -o StrictHostKeyChecking=no ec2-user@65.2.37.229 \"mkdir -p /tmp/frontend\""
                    bat "scp -o StrictHostKeyChecking=no -r dist/revconnect-ui/browser/* ec2-user@65.2.37.229:/tmp/frontend/"
                    bat "ssh -o StrictHostKeyChecking=no ec2-user@65.2.37.229 \"sudo rm -rf /var/www/html/revconnect-ui/browser/* && sudo mkdir -p /var/www/html/revconnect-ui/browser/ && sudo cp -r /tmp/frontend/* /var/www/html/revconnect-ui/browser/ && sudo chown -R ec2-user:ec2-user /var/www/html/revconnect-ui && sudo systemctl restart nginx\""
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed RevConnect Frontend"
        }
        failure {
            echo "Deployment failed! Check the Jenkins logs."
        }
    }
}
