pipeline {
    agent any

    environment {
        PROJECT_NAME = "Java-springboot-jenkins-terraform"
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
        PROJECT_DIR = "JtProject"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: "${GITHUB_REPO_URL}", credentialsId: 'terraform'
                echo "✅ Code checkout completed"
            }
        }

        stage('Build') {
            steps {
                echo "🔨 Building Project..."
                dir("${PROJECT_DIR}") {
                    sh 'mvn clean package -DskipTests'
                }
                echo "✅ Build completed"
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Running Tests..."
                dir("${PROJECT_DIR}") {
                    sh 'mvn test'
                }
                echo "✅ Tests completed"
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo "📦 Archiving artifacts..."
                dir("${PROJECT_DIR}") {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
                echo "✅ Artifacts archived"
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying Application..."
                dir("${PROJECT_DIR}") {
                    sh '''
                        echo "Deployment ready"
                        ls -l target/*.jar
                    '''
                }
                echo "✅ Deployment completed"
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning workspace..."
            cleanWs()
        }
        success {
            echo '🎉 Pipeline succeeded! All stages completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed! Check the logs above.'
        }
    }
}
