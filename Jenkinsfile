pipeline {
    agent any

    environment {
        EC2_IP = '18.141.234.116'
        EC2_USER = 'ec2-user'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
            }
        }

        stage('Deploy to EC2') {
    steps {
        sshagent(credentials: ['ec2-ssh-key']) {
            sh """
                scp -o StrictHostKeyChecking=no index.html ${EC2_USER}@${EC2_IP}:/tmp/index.html
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                    sudo mv /tmp/index.html /usr/share/nginx/html/index.html
                    echo "Deployed successfully!"
                '
            """
        }
    }
}
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
