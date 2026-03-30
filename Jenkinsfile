pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        APP_SERVER = "YOUR_VM2_IP"
        APP_USER = "YOUR_VM2_USER"
        IMAGE_NAME = "cicd-rhel96-demo"
        CONTAINER_NAME = "cicd-rhel96-demo"
        SSH_CREDENTIALS_ID = "vm2-ssh-key"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    source venv/bin/activate
                    python -m pytest -q
                '''
            }
        }

        stage('Build Image') {
    steps {
        sh 'sudo podman build -t cicd-rhel96-demo:latest .'
    }
}

stage('Deploy to App Server') {
    steps {
        sshagent(credentials: ["vm2-ssh-key"]) {
            sh '''
                sudo podman save -o /tmp/cicd-rhel96-demo.tar cicd-rhel96-demo:latest
                scp -o StrictHostKeyChecking=no /tmp/cicd-rhel96-demo.tar ${APP_USER}@${APP_SERVER}:/tmp/cicd-rhel96-demo.tar
                ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER} "
                    podman load -i /tmp/cicd-rhel96-demo.tar &&
                    podman stop cicd-rhel96-demo || true &&
                    podman rm cicd-rhel96-demo || true &&
                    podman run -d --name cicd-rhel96-demo -p 5000:5000 cicd-rhel96-demo:latest
                "
            '''
        }
    }
}
    }
}
