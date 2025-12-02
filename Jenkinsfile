pipeline {
    agent any

    environment {
        PROJECT_NAME = "E-commerce-project-springBoot"
        GITHUB_REPO_URL = "https://github.com/AishwaryaPawar149/Java-springboot-jenkins-terraform.git"

        # EC2 details
        EC2_USER = "ubuntu"
        EC2_IP   = "65.1.86.61"
        EC2_APP_DIR = "/home/ubuntu/app"

        # Database details (RDS)
        DB_NAME = "ecommjava"
        DB_USER = "admin"
        DB_PASS = "Admin123"  // valid password
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
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Prepare application.properties') {
            steps {
                sh """
                mkdir -p deploy
                echo "spring.datasource.url=jdbc:mysql://${DB_ENDPOINT}/${DB_NAME}?createDatabaseIfNotExist=true" > deploy/application.properties
                echo "spring.datasource.username=${DB_USER}" >> deploy/application.properties
                echo "spring.datasource.password=${DB_PASS}" >> deploy/application.properties
                echo "spring.jpa.hibernate.ddl-auto=update" >> deploy/application.properties
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                # Create app directory on EC2
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "mkdir -p ${EC2_APP_DIR}"

                # Copy JAR and application.properties
                scp -o StrictHostKeyChecking=no target/*.jar ${EC2_USER}@${EC2_IP}:${EC2_APP_DIR}/app.jar
                scp -o StrictHostKeyChecking=no deploy/application.properties ${EC2_USER}@${EC2_IP}:${EC2_APP_DIR}/application.properties

                # Install Java if missing & setup systemd service
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} <<'ENDSSH'
                sudo yum install java-11-openjdk -y || true

                sudo bash -c 'cat > /etc/systemd/system/springboot-app.service <<EOL
[Unit]
Description=Spring Boot E-commerce App
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/app
ExecStart=/usr/bin/java -jar /home/ubuntu/app/app.jar
SuccessExitStatus=143
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOL'

                sudo systemctl daemon-reload
                sudo systemctl enable springboot-app
                sudo systemctl restart springboot-app
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
        }
        failure {
            echo 'Pipeline failed! Check logs.'
        }
    }
}
