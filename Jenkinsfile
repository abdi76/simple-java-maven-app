
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'abdi76/my-java-app'  // 🔹 Replace with your Docker Hub username and image name (e.g., 'mydockeruser/my-java-app')
        DOCKER_TAG = 'latest'  // 🔹 Change to a version if needed (e.g., 'v1.0.0')
        DEPLOY_SERVER = 'ubuntu@18.175.168.22'  // 🔹 Replace with your EC2 instance public IP (e.g., 'ubuntu@54.12.34.56')
    }

    stages {
        stage('Clone Repository') {
            steps { 
                git branch: 'main', url: 'https://github.com/abdi76/End-End-Jenkins-CICD.git'  
                // 🔹 Replace 'your-github' with your GitHub username or organization (e.g., 'my-github-user')
                // 🔹 Ensure the repo URL is correct before running
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'  // 🔹 Cleans, compiles, and packages the Java application
                sh 'mvn test'  // 🔹 Runs unit tests to verify the build
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'  
                // 🔹 Builds the Docker image using the Dockerfile in the project root

                withCredentials([string(credentialsId: 'docker-token-cred', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login -u abdi76 --password-stdin'  
                    // 🔹 Authenticates to Docker Hub using Jenkins stored credentials (Replace 'docker-hub-token' with the actual credentials ID)
                }

                sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'  
                // 🔹 Pushes the Docker image to Docker Hub
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {  
                    // 🔹 Uses Jenkins SSH credentials (Replace 'deployment-server-key' with actual Jenkins credentials ID for EC2)
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER << EOF
                    docker pull $DOCKER_IMAGE:$DOCKER_TAG  # 🔹 Pulls the latest Docker image from Docker Hub

                    docker stop sample-app || true  # 🔹 Stops any existing running container (if exists)
                    docker rm sample-app || true  # 🔹 Removes the stopped container to free up the name

                    docker run -d -p 8080:8080 --name sample-app $DOCKER_IMAGE:$DOCKER_TAG  
                    # 🔹 Runs the new Docker container and exposes it on port 8080
                    EOF
                    '''
                }
            }
        }
    }
}
