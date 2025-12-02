pipeline {
    agent any

    environment {
        // Project details
        PROJECT_NAME = "E-commerce-project-springBoot"
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"
        
        // Spring Boot project directory
        APP_DIR = "JtProject"  // The subdirectory containing pom.xml

        // EC2 details - Amazon Linux with ec2-user
        EC2_USER = "ec2-user"  // Changed from ubuntu to ec2-user
        EC2_IP   = "65.0.93.50"
        EC2_APP_DIR = "/home/ec2-user/app"  // Changed path for ec2-user

        // Database details (RDS)
        DB_NAME = "ecommjava"
        DB_USER = "admin"
        DB_PASS = "Admin123"
        DB_ENDPOINT = "terraform-20251202122328798100000001.cdwkuiksmrsm.ap-south-1.rds.amazonaws.com:3306"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${GITHUB_REPO_URL}", branch: 'master'
            }
        }

        stage('Build') {
            steps {
                dir("${APP_DIR}") {  // Change to the Spring Boot project directory
                    sh 'mvn clean package'
                }
            }
        }

        stage('Test') {
            steps {
                dir("${APP_DIR}") {  // Change to the Spring Boot project directory
                    sh 'mvn test'
                }
            }
        }

        stage('Prepare application.properties') {
            steps {
                dir("${APP_DIR}") {  // Stay in the project directory
                    sh """
                    mkdir -p deploy
                    echo "spring.datasource.url=jdbc:mysql://${DB_ENDPOINT}/${DB_NAME}?createDatabaseIfNotExist=true" > deploy/application.properties
                    echo "spring.datasource.username=${DB_USER}" >> deploy/application.properties
                    echo "spring.datasource.password=${DB_PASS}" >> deploy/application.properties
                    echo "spring.jpa.hibernate.ddl-auto=update" >> deploy/application.properties
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                # Create app directory on EC2
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "mkdir -p ${EC2_APP_DIR}"

                # Copy JAR and application.properties from the correct directory
                scp -o StrictHostKeyChecking=no ${APP_DIR}/target/*.jar ${EC2_USER}@${EC2_IP}:${EC2_APP_DIR}/app.jar
                scp -o StrictHostKeyChecking=no ${APP_DIR}/deploy/application.properties ${EC2_USER}@${EC2_IP}:${EC2_APP_DIR}/application.properties

                # Install Java and setup systemd service on Amazon Linux
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} <<'ENDSSH'
                # Install Java 11 on Amazon Linux
                sudo yum install java-11-amazon-corretto -y || sudo yum install java-11-openjdk -y

                # Create systemd service for Spring Boot app
                sudo bash -c 'cat > /etc/systemd/system/springboot-app.service <<EOL
[Unit]
Description=Spring Boot E-commerce App
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/app
ExecStart=/usr/bin/java -jar /home/ec2-user/app/app.jar --spring.config.location=/home/ec2-user/app/application.properties
SuccessExitStatus=143
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOL'

                # Reload systemd, enable and start the service
                sudo systemctl daemon-reload
                sudo systemctl enable springboot-app
                sudo systemctl restart springboot-app
                
                # Check service status
                sudo systemctl status springboot-app --no-pager
ENDSSH
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded! Application deployed on EC2.'
            echo "Access your application at: http://${EC2_IP}:8080"
        }
        failure {
            echo 'Pipeline failed! Check logs.'
        }
    }
}