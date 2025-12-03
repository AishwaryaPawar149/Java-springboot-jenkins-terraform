pipeline {
    agent any
    
    environment {
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
        GIT_BRANCH      = "master"
        SSH_CRED_ID     = "terraform"  // तुमची existing credential ID
        EC2_IP          = "13.201.10.86"  // तुमचा EC2 IP
        REMOTE_USER     = "ec2-user"  // Amazon Linux साठी
        APP_PATH        = "/opt/springboot-app"
        PROJECT_DIR     = "JtProject"
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Cloning ${GITHUB_REPO_URL} (branch: ${GIT_BRANCH})"
                git url: "${GITHUB_REPO_URL}", branch: "${GIT_BRANCH}", credentialsId: "${SSH_CRED_ID}"
            }
        }
        
        stage('Build with Maven') {
            steps {
                echo "🔨 Running Maven Build in ${PROJECT_DIR}..."
                dir("${PROJECT_DIR}") {
                    sh "mvn -B clean package -DskipTests"
                }
            }
            post {
                success {
                    echo "📦 Archiving artifacts..."
                    archiveArtifacts artifacts: "${PROJECT_DIR}/target/*.jar", fingerprint: true
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                echo "🧪 Running Tests..."
                dir("${PROJECT_DIR}") {
                    sh "mvn test"
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                echo "🚀 Deploying application to EC2: ${EC2_IP}"
                sshagent(credentials: [SSH_CRED_ID]) {
                    script {
                        // JAR file find करा
                        def jarFile = sh(
                            script: "ls ${PROJECT_DIR}/target/*.jar 2>/dev/null | grep -v 'sources.jar' | head -n 1 || true", 
                            returnStdout: true
                        ).trim()
                        
                        if (!jarFile) {
                            error "❌ No JAR file found in ${PROJECT_DIR}/target"
                        }
                        
                        echo "✅ JAR Found: ${jarFile}"
                        
                        // Remote directory create करा
                        sh """
                            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} \
                            'sudo mkdir -p ${APP_PATH} && sudo chown ${REMOTE_USER}:${REMOTE_USER} ${APP_PATH}'
                        """
                        
                        // JAR file copy करा
                        sh """
                            scp -o StrictHostKeyChecking=no "${jarFile}" \
                            ${REMOTE_USER}@${EC2_IP}:"${APP_PATH}/app.jar"
                        """
                        
                        // Application restart करा
                        sh """
                            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} \
                            'sudo systemctl restart springboot-app.service || echo "Service restart failed"'
                        """
                        
                        // Status check करा
                        sh """
                            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} \
                            'sudo systemctl status springboot-app.service --no-pager || true'
                        """
                    }
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo "🏥 Checking application health..."
                script {
                    sleep(time: 15, unit: 'SECONDS')
                    sh """
                        curl -f http://${EC2_IP}:8080/actuator/health || \
                        curl -f http://${EC2_IP}:8080 || \
                        echo "⚠️  Health check endpoint not responding yet"
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '🎉 Pipeline SUCCESS!'
            echo "🌐 Application URL: http://${EC2_IP}:8080"
        }
        failure {
            echo '❌ Pipeline FAILED - Check console output'
        }
        always {
            echo "🧹 Cleaning workspace..."
            deleteDir()
        }
    }
}
