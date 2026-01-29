## Flow

Developer pushes Python code to GitHub

Jenkins pulls the code

Jenkins runs tests

Jenkins performs SonarQube code analysis

Jenkins builds Docker image

Docker image is pushed to Docker Hub

## 🧰 Tools Used

Python – Sample application

GitHub – Source control

Jenkins – CI/CD orchestration

SonarQube – Code quality & security

Docker – Containerization

## 📁 Step 1: Create Python Sample App

### Project Structure
```
ci-cd-python-sonar/
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
└── sonar-project.properties
```

## app.py
```
def add(a, b):
    return a + b

if __name__ == "__main__":
    print(add(2, 3))
```

## test_app.py
```
from app import add

def test_add():
    assert add(2, 3) == 5
```
## requirements.txt
```
pytest
```

## 🌐 Step 2: Push Code to GitHub
```
git init
git add .
git commit -m "Initial CI/CD POC with SonarQube"
git branch -M main
git remote add origin https://github.com/<username>/ci-cd-python-sonar.git
git push -u origin main
```

## 🐳 Step 3: Create Dockerfile
```
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

## 🔍 Step 4: SonarQube Configuration

### Run SonarQube (Docker – easiest)
```
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
```

### Access:
👉 http://localhost:9000

Default login: admin / admin
```
sonar-project.properties
sonar.projectKey=python-ci-cd-poc
sonar.projectName=Python CI CD POC
sonar.projectVersion=1.0
sonar.sources=.
sonar.language=py
sonar.python.version=3
```

### Generate SonarQube Token:
```
SonarQube → My Account → Security → Generate Token
```

## ⚙️ Step 5: Jenkins Setup
Install Jenkins Plugins

Git

Pipeline

Docker Pipeline

SonarQube Scanner

### Configure SonarQube in Jenkins
```
Manage Jenkins → System → SonarQube Servers
Name: SonarQube
URL: http://localhost:9000
Token: (Sonar token)
```

## 📜 Step 6: Jenkinsfile (Pipeline as Code)
```
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhub/python-ci-cd"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/<username>/ci-cd-python-sonar.git'
            }
        }

        stage('Install Dependencies & Test') {
            steps {
                sh '''
                pip install -r requirements.txt
                pytest
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
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
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
```

## 🔐 Step 7: Jenkins Credentials

Add credentials in Jenkins:

Docker Hub username/password

ID: dockerhub-creds

## 🚀 Step 8: Run the Pipeline

Create Pipeline Job in Jenkins

Select Pipeline from SCM

Choose Git & repo URL

Click Build Now

## ✅ Final Output
```
✔ Code tested with pytest
✔ SonarQube quality gate report
✔ Docker image built
✔ Image pushed to Docker Hub
```
