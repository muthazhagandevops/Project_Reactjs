pipeline { 
    agent any

    environment {
        DOCKER_IMAGE_NAME_DEV = 'muthazhagan/dev:latest'  // Docker image name for dev branch
        DOCKER_IMAGE_NAME_PROD = 'muthazhagan/prod:latest'  // Docker image name for prod branch
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Get the branch that triggered the build
                    def branch = env.GIT_BRANCH ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()

                    // Checkout the appropriate branch based on the trigger
                    echo "Checking out branch: ${branch}"
                    if (branch == "main") {
                        echo "Checked out 'main' branch"
                        git branch: 'main', url: 'https://github.com/muthazhagandevops/Project_Reactjs.git'
                    } else {
                        echo "Checked out 'dev' branch"
                        git branch: 'dev', url: 'https://github.com/muthazhagandevops/Project_Reactjs.git'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image from the checked out branch"
                    // Run the build script to build the Docker image
                    sh './build.sh'

                    // After building the image, tag it for Docker Hub
                    echo "Tagging Docker image as ${DOCKER_IMAGE_NAME_DEV}"
                    sh "docker tag project_reactjs_dev_web:latest $DOCKER_IMAGE_NAME_DEV"
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub using the credentials securely
                    echo "Logging into Docker Hub"
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    }

                    // Push the image to the correct Docker repository
                    echo "Pushing Docker image to 'dev' repository"
                    sh "docker push $DOCKER_IMAGE_NAME_DEV"
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying to the server"
                    // Run the deployment script to deploy the application
                    sh './deploy.sh'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment succeeded!'
        }
        failure {
            echo 'Build or deployment failed!'
        }
    }
}

