pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kavindu/task-manager:latest"         // Change 'kavindu' to your DockerHub username
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"      // Jenkins credentials for DockerHub
        SSH_CREDENTIALS_ID = "deploy-server"                 // Jenkins credentials ID for remote server SSH
        SSH_TARGET = "ubuntu@54.254.18.85"                   // Replace with your server IP or hostname
        DOCKER_CONTAINER = "task-manager-app"                // Name of the running container
    }

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                echo " Checking out source code..."
                git branch: 'main', 
                    url: 'https://github.com/KL193/ci-cd-project-task.git' 
            }
        }

        stage('Build') {
            steps {
                echo " Building Spring Boot application..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Running tests..."
                sh 'mvn test'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    echo "🐳 Building and pushing Docker image..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                        docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo " Deploying to remote server..."
                    sshagent([SSH_CREDENTIALS_ID]) {
                        sh '''
                        ssh -o StrictHostKeyChecking=no $SSH_TARGET << EOF
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker pull $DOCKER_IMAGE
                        docker stop $DOCKER_CONTAINER || true
                        docker rm $DOCKER_CONTAINER || true
                        docker run -d --name $DOCKER_CONTAINER -p 8080:8080 $DOCKER_IMAGE
                        docker logout
                        EOF
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            emailext (
                to: 'kavindu@example.com',
                subject: " SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>Good news! The job <b>${env.JOB_NAME}</b> build <b>${env.BUILD_NUMBER}</b> succeeded.</p>""",
                replyTo: 'noreply@example.com',
                from: 'noreply@example.com'
            )
        }

        failure {
            emailext (
                to: 'kavindu@example.com',
                subject: " FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>Unfortunately, the job <b>${env.JOB_NAME}</b> build <b>${env.BUILD_NUMBER}</b> failed.</p>""",
                replyTo: 'noreply@example.com',
                from: 'noreply@example.com'
            )
        }
    }
}
