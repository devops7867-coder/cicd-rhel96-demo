pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        APP_SERVER = "192.168.56.104"
        APP_USER = "root"
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
                    . venv/bin/activate
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    python -m pytest -q
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    sudo podman build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    sh '''
                        ARCHIVE="${WORKSPACE}/${IMAGE_NAME}-${BUILD_NUMBER}.tar"

                        sudo podman save --format oci-archive -o "$ARCHIVE" ${IMAGE_NAME}:latest

                        scp -o StrictHostKeyChecking=no "$ARCHIVE" ${APP_USER}@${APP_SERVER}:/tmp/${IMAGE_NAME}.tar

                        ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER} "
                            podman load -i /tmp/${IMAGE_NAME}.tar &&
                            podman stop ${CONTAINER_NAME} || true &&
                            podman rm ${CONTAINER_NAME} || true &&
                            podman run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE_NAME}:latest
                        "

                        rm -f "$ARCHIVE"
                    '''
                }
            }
        }
    }
}
