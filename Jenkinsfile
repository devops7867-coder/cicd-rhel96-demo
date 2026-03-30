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
                sh '''
                    podman build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    sh '''
                        podman save -o /tmp/${IMAGE_NAME}.tar ${IMAGE_NAME}:latest
                        scp -o StrictHostKeyChecking=no /tmp/${IMAGE_NAME}.tar ${APP_USER}@${APP_SERVER}:/tmp/${IMAGE_NAME}.tar
                        ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER} "
                            podman load -i /tmp/${IMAGE_NAME}.tar &&
                            podman stop ${CONTAINER_NAME} || true &&
                            podman rm ${CONTAINER_NAME} || true &&
                            podman run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE_NAME}:latest
                        "
                    '''
                }
            }
        }
    }
}
