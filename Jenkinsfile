pipeline {
    agent any

    environment {
        // AWS EC2 Details
        EC2_USER = 'ec2-user'
        EC2_IP = '65.2.37.229'
        SSH_KEY_ID = 'aws-ec2-ssh-key' // Jenkins Credentials ID holding rc-key.pem
        DEST_DIR = '/var/www/html/revconnect-ui/browser/'
    }

    tools {
        nodejs 'node' // Requires NodeJS plugin configured in Jenkins Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('revconnect-app') { // Adjust this depending on repo structure
                    sh 'npm install --legacy-peer-deps'
                }
            }
        }

        stage('Build Angular App') {
            steps {
                dir('revconnect-app') {
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ["${SSH_KEY_ID}"]) {
                    dir('revconnect-app') {
                        // Create a temporary directory on the EC2 instance
                        sh "ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} 'mkdir -p /tmp/frontend'"
                        
                        // Copy built files to the temporary directory
                        sh "scp -o StrictHostKeyChecking=no -r dist/revconnect-ui/browser/* ${EC2_USER}@${EC2_IP}:/tmp/frontend/"
                        
                        // Move files to NGINX web root and restart NGINX
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                            sudo rm -rf ${DEST_DIR}* &&
                            sudo mkdir -p ${DEST_DIR} &&
                            sudo cp -r /tmp/frontend/* ${DEST_DIR} &&
                            sudo chown -R ec2-user:ec2-user /var/www/html/revconnect-ui &&
                            sudo systemctl restart nginx
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed RevConnect Frontend to http://${EC2_IP}"
        }
        failure {
            echo 'Deployment failed! Check the Jenkins logs.'
        }
    }
}
