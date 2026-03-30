pipeline {
    agent any

    environment {
        APP_SERVER = "192.168.56.104"
        USER = "root"
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
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                podman build -t cicd-app .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                podman save cicd-app -o /tmp/app.tar
                scp /tmp/app.tar ${USER}@${APP_SERVER}:/tmp/
                
                ssh ${USER}@${APP_SERVER} "
                    podman load -i /tmp/app.tar &&
                    podman stop app || true &&
                    podman rm app || true &&
                    podman run -d -p 5000:5000 --name app cicd-app
                "
                '''
            }
        }
    }
}
