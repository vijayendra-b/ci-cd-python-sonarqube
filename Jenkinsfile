pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "vijayendra1/ci-cd-python-sonarqube"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/vijayendra-b/ci-cd-python-sonarqube.git'
            }
        }

        stage('Install Dependencies & Test') {
            steps {
                sh '''
                python3 --version
                pip3 --version
                pip3 install -r requirements.txt
                python3 -m pytest
                '''
            }
        }


        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'b799765c-9aab-4075-8537-541d450e7f9d',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }
    }
}
