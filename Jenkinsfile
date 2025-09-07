
pipeline {
    agent any

    tools {
        maven '3.6.3' // Make sure this matches the configured Maven version in Jenkins
    }

    environment {
        MAVEN_OPTS = "-Dmaven.repo.local=.m2/repository"
        IMAGE_NAME = "jyotirmoy43/myimg2"
        CONTAINER_NAME = "c${env.BUILD_NUMBER}"
        APP_PORT = "8081"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Checking out code from GitHub...'
                git url: 'https://github.com/jyotirmoy43/insurenc-project.git'
            }
        }

        stage('Build & Test') {
            steps {
                echo '🔨 Compiling and testing the code...'
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Code Quality - Checkstyle') {
            steps {
                echo '🧹 Running Checkstyle...'
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo '📤 Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                echo '🚀 Running Docker container...'
                sh '''
                    # Stop any container already using the host port (optional clean-up)
                    OLD_CONTAINER=$(docker ps -q --filter "publish=$APP_PORT")
                    if [ ! -z "$OLD_CONTAINER" ]; then
                        echo "🛑 Stopping old container using port $APP_PORT..."
                        docker stop $OLD_CONTAINER
                        docker rm $OLD_CONTAINER
                    fi

                    # Run new container
                    docker run -dt -p $APP_PORT:$APP_PORT --name $CONTAINER_NAME $IMAGE_NAME
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment completed successfully!'
            echo "🌐 App should be available at http://<YOUR-SERVER-IP>:${APP_PORT}"
        }
        failure {
            echo '❌ Build or deployment failed. Check the logs.'
        }
    }
}
