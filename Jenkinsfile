pipeline {
    agent any

    environment {
        PROJECT_NAME = "Java-springboot-jenkins-terraform"
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
    }

    stages {

        stage('Checkout Code') {
            steps {
                // Using credentials 'terraform' (make sure it exists in Jenkins)
                git branch: 'master', url: "${GITHUB_REPO_URL}", credentialsId: 'terraform'
            }
        }

        stage('Build') {
            steps {
                echo "Building Project..."
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                echo "Running Tests..."
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying Application..."
                sh '''
                # TODO: Add deploy commands here
                # Example (if using EC2):
                # scp -o StrictHostKeyChecking=no target/*.jar ubuntu@<EC2-IP>:/home/ubuntu/
                # ssh ubuntu@<EC2-IP> "nohup java -jar /home/ubuntu/*.jar &"
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        success {
            echo '🎉 Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
