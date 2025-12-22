pipeline {
    agent any

    environment {
        // REPLACE WITH YOUR DOCKER HUB USERNAME
        DOCKER_USER = 'aminerahbani' 
        // ID of the credentials you created in Jenkins
        DOCKER_CREDS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                // Gets the code from your Git repo
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo '--- Building Images with Compose ---'
                    // Builds images as defined in docker-compose.yml
                    sh 'docker-compose build'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    echo '--- Logging in ---'
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    echo '--- Pushing Images ---'
                    sh 'docker-compose push'
                }
            }
        }

        // --- THIS IS THE MISSING PART ---
        stage('Deploy to Production') {
            steps {
                script {
                    echo '--- Deploying App ---'
                    // Stops old containers to free up ports
                    sh 'docker-compose down || true'
                    // Starts the new containers in the background
                    sh 'docker-compose up -d'
                }
            }
        }
        // --------------------------------
    }

    post {
        always {
            // Logout for security
            sh 'docker logout'
        }
        success {
            echo 'Build, Push, and Deploy Successful!'
        }
    }
}
