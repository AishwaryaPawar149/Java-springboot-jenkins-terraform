pipeline {
    agent any

    environment {
        PROJECT_NAME = "Java-springboot-jenkins-terraform"
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: "${GITHUB_REPO_URL}", credentialsId: 'terraform'
                
                // Debug information
                sh '''
                    echo "Current directory:"
                    pwd
                    echo "\nListing files:"
                    ls -la
                    echo "\nSearching for pom.xml:"
                    find . -name "pom.xml" -type f
                '''
            }
        }

        stage('Build') {
            steps {
                echo "Building Project..."
                script {
                    // Find and navigate to directory with pom.xml
                    def pomDir = sh(script: 'find . -name "pom.xml" -type f -exec dirname {} \\; | head -1', returnStdout: true).trim()
                    if (pomDir) {
                        dir(pomDir) {
                            sh 'mvn clean package'
                        }
                    } else {
                        error "pom.xml not found in repository"
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running Tests..."
                script {
                    def pomDir = sh(script: 'find . -name "pom.xml" -type f -exec dirname {} \\; | head -1', returnStdout: true).trim()
                    dir(pomDir) {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying Application..."
                sh '''
                # TODO: Add deploy commands
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
