pipeline {
    agent any

    environment {
        // 🔹 Update with your Docker Hub username and image name
        DOCKER_IMAGE = 'kiranitth/my-java-app'  // Example: 'mydockeruser/my-java-app'
        DOCKER_TAG = 'latest'  // Change if you want to version your images (e.g., 'v1.0.0')

        // 🔹 Update with your EC2 instance's public IP or domain name
        DEPLOY_SERVER = 'ubuntu@54.173.210.219'  // Example: 'ubuntu@54.12.34.56'
    }

    stages {
        stage('Clone Repository') {
            steps { 
                // 🔹 Replace with your actual GitHub repo URL and branch
                git branch: 'main', url: 'https://github.com/kirankumaritth/simple-java-maven-app.git'  
                // Example: 'https://github.com/my-github-user/sample-java-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                // 🔹 Maven build and unit tests
                sh 'mvn clean package'
                sh 'mvn test'
            }
        }

        stage('Docker Build & Push') {
            steps {
                // 🔹 Build the Docker image with the specified tag
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'

                // 🔹 Use Jenkins credentials for secure Docker Hub login
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login -u your-dockerhub-username --password-stdin'
                }

                // 🔹 Push the Docker image to Docker Hub
                sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'
            }
        }

        stage('Deploy to EC2') {
            steps {
                // 🔹 Use SSH agent credentials to connect to EC2
                sshagent(['deployment-server-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER << EOF

                    # 🔹 Pull the latest Docker image
                    docker pull $DOCKER_IMAGE:$DOCKER_TAG

                    # 🔹 Stop and remove any existing container with the same name
                    docker stop sample-app || true
                    docker rm sample-app || true

                    # 🔹 Run the new container on port 8080
                    docker run -d -p 8080:8080 --name sample-app $DOCKER_IMAGE:$DOCKER_TAG

                    EOF
                    '''
                }
            }
        }
    }
}
