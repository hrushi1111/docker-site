
# 🚀 CI/CD with Jenkins & Docker

This project demonstrates how to set up a **basic Jenkins pipeline** to build and deploy an application using **Jenkins + Docker**.

---

## 📌 Objective

Automate the process of:

1. Pulling the latest code from GitHub
2. Building the application
3. Running tests (if available)
4. Building a Docker image
5. Pushing the image to **DockerHub**
6. Deploying the container

---

## 🛠 Tools Used

* **Jenkins** – Automation server for CI/CD
* **Docker** – Containerization
* **GitHub** – Source code management
* **Your App** – Any web or backend application

---

## 📂 Project Structure

```
my-app/
├── Dockerfile
├── Jenkinsfile
├── src/           # Application source code
└── README.md
```

---

## ⚙️ Setup Instructions

### 1. Install Jenkins

* Install Jenkins locally or on a cloud VM.
* Access Jenkins at: `http://<server-ip>:8080`

### 2. Install Docker

* Install Docker on the Jenkins host:

  ```bash
  sudo apt install docker.io -y
  sudo usermod -aG docker jenkins
  sudo systemctl restart jenkins
  ```

### 3. Configure Jenkins

1. Create a new **Pipeline job** in Jenkins.
2. Set it to use **Pipeline script from SCM** → Git → your repo URL.
3. Add **DockerHub credentials** in Jenkins (ID: `dockerhub-creds`).

### 4. Connect GitHub to Jenkins

* Add a webhook in your GitHub repo →
  `http://<jenkins-server>:8080/github-webhook/`
* Trigger: On **push events**

---

## 📜 Jenkinsfile (Generic Example)

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/my-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building application..."'
                // Example: mvn clean package OR ./gradlew build OR make build
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Running tests..."'
                // Example: mvn test OR pytest tests/
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop my-app || true
                docker rm my-app || true
                docker run -d --name my-app -p 8080:8080 $DOCKER_IMAGE:latest
                '''
            }
        }
    }
}
```

---

## ▶️ How It Works

1. **Checkout** – Jenkins pulls the latest code from GitHub
2. **Build** – Application is built using your build tool (Maven, Gradle, etc.)
3. **Test** – Runs your project tests (unit/integration)
4. **Docker Build & Push** – Builds and pushes Docker image to DockerHub
5. **Deploy** – Runs the app as a Docker container

---

## ✅ Deliverables

* Jenkins pipeline (`Jenkinsfile`)
* Dockerized application
* Automatic build, test, push, and deploy on each GitHub commit
