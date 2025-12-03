pipeline {
    agent any

    environment {
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
        GIT_BRANCH      = "master"
        SSH_CRED_ID     = "terraform"
        EC2_IP          = "13.201.56.92"
        REMOTE_USER     = "ubuntu"
        JAR_NAME        = "application.jar"
    }

    stages {
        
        stage('Clean Workspace') {
            steps {
                echo "🧹 Cleaning old workspace..."
                cleanWs()  // Purani files delete karaycha
            }
        }
        
        stage('Checkout Code') {
            steps {
                echo "📥 Cloning fresh code from GitHub..."
                git branch: "${GIT_BRANCH}", url: "${GITHUB_REPO_URL}"
            }
        }

        stage('Set Executable Permission') {
            steps {
                echo "🔑 Setting executable permission for mvnw..."
                sh 'chmod +x mvnw'
            }
        }
        
        stage('Build Project') {
            steps {
                echo "🔨 Building Spring Boot application..."
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Rename Jar') {
            steps {
                echo "📦 Renaming JAR file..."
                sh "cp target/*.jar ${JAR_NAME}"
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                echo "🚀 Deploying to EC2 instance..."
                sshagent(credentials: ["${SSH_CRED_ID}"]) {
                    sh """
                        echo "📤 Copying JAR to EC2..."
                        scp -o StrictHostKeyChecking=no ${JAR_NAME} ${REMOTE_USER}@${EC2_IP}:/home/ubuntu/
                        
                        echo "🔄 Restarting application on EC2..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} << 'EOF'
                            # Purani application stop kara
                            pkill -f ${JAR_NAME} || true
                            
                            # Navin application start kara
                            nohup java -jar /home/ubuntu/${JAR_NAME} > app.log 2>&1 &
                            
                            echo "✅ Application deployed successfully!"
EOF
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "✅ Pipeline completed successfully! 🎉"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
    }
}
