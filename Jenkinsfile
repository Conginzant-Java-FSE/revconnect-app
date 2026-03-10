pipeline {
    agent any

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
                withCredentials([sshUserPrivateKey(credentialsId: 'aws-ec2-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    bat 'ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@65.2.37.229 "mkdir -p /tmp/frontend"'
                    bat 'scp -o StrictHostKeyChecking=no -i "%SSH_KEY%" -r dist/revconnect-ui/browser/* %SSH_USER%@65.2.37.229:/tmp/frontend/'
                    bat 'ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@65.2.37.229 "sudo rm -rf /var/www/html/revconnect-ui/browser/*; sudo mkdir -p /var/www/html/revconnect-ui/browser/; sudo cp -r /tmp/frontend/* /var/www/html/revconnect-ui/browser/; sudo chown -R ec2-user:ec2-user /var/www/html/revconnect-ui; sudo systemctl restart nginx"'
                }
            }
        }
    }

    post {
        always {
            echo 'Deployment Pipeline Finished.'
        }
    }
}
