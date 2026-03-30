pipeline {
    agent any

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
                rm -f /tmp/cicd-rhel96-demo.tar
                sudo podman save --format oci-archive -o /tmp/cicd-rhel96-demo.tar cicd-rhel96-demo:latest
                scp -o StrictHostKeyChecking=no /tmp/cicd-rhel96-demo.tar devops@192.168.56.20:/tmp/cicd-rhel96-demo.tar
                ssh -o StrictHostKeyChecking=no devops@192.168.56.20 "
                    podman load -i /tmp/cicd-rhel96-demo.tar &&
                    podman stop cicd-rhel96-demo || true &&
                    podman rm cicd-rhel96-demo || true &&
                    podman run -d --name cicd-rhel96-demo -p 5000:5000 cicd-rhel96-demo:latest
                "
            '''
        }
    }
}    }
}
