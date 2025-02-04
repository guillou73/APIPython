pipeline {
    agent any
    environment {
        VENV_DIR = 'venv'
        DOCKER_IMAGE = 'guillou73/flask-api:latest'
        FLASK_APP_PORT = '5310'
        SERVER_IP = '18.132.73.146' // Replace with your server's public IP
    }
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/guillou73/APIPython.git', branch: 'main'
            }
        }
        stage('Set Up Virtual Environment') {
            steps {
                script {
                    // Create a virtual environment
                    sh 'python3 -m venv ${VENV_DIR}'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    // Activate virtual environment, upgrade pip, and install dependencies
                    sh '''
                        source ${VENV_DIR}/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Activate virtual environment and run pytest
                    sh '''
                        source ${VENV_DIR}/bin/activate
                        pytest test_app.py --junitxml=test-results.xml
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        sh 'echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin'
                    }
                    // Push the image
                    sh 'docker push ${DOCKER_IMAGE}'
                }
            }
        }
    }
    post {
        success {
            // Output the full URL to access the Flask API
            echo "Build succeeded. The Flask API is running at http://${SERVER_IP}:${FLASK_APP_PORT}/data"
            // Send success email notification
            emailext subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}", 
                     body: "The build was successful. View details at ${env.BUILD_URL}", 
                     recipientProviders: [[$class: 'DevelopersRecipientProvider']]
        }
        failure {
            // Send failure email notification
            emailext subject: "Jenkins Build FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}", 
                     body: "The build has failed. View details at ${env.BUILD_URL}", 
                     recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            // Additional failure alert
            mail to: 'guyseutcheu@gmail.com', 
                 subject: "Jenkins Build FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}", 
                 body: "The build has failed. Please check the Jenkins logs for details: ${env.BUILD_URL}"
        }
    }
}
