pipeline {
    agent any

    environment {
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
        GIT_BRANCH      = "master"
        SSH_CRED_ID     = "terraform"
        EC2_IP          = "13.201.56.92"   // Updated IP
        REMOTE_USER     = "ubuntu"
        JAR_NAME        = "application.jar"
    }

    stages {
        
        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GITHUB_REPO_URL}"
            }
        }
        
        stage('Build Project') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Rename Jar') {
            steps {
                sh "cp target/*.jar ${JAR_NAME}"
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ["${SSH_CRED_ID}"]) {
                    sh """
                        scp -o StrictHostKeyChecking=no ${JAR_NAME} ${REMOTE_USER}@${EC2_IP}:/home/ubuntu/
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} << EOF
                            pkill -f ${JAR_NAME} || true
                            nohup java -jar /home/ubuntu/${JAR_NAME} > app.log 2>&1 &
                        EOF
                    """
                }
            }
        }
    }
}
