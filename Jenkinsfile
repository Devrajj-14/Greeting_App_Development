pipeline {
    agent any

    environment {
        JAR_NAME        = 'GreetingApp-0.0.1-SNAPSHOT.jar'
        EC2_USER        = 'ubuntu'
        EC2_HOST        = '13.51.195.17'
        DEPLOY_DIR      = '/home/ubuntu'
        APP_PORT        = '9090'
        DB_URL          = credentials('DB_URL')
        DB_USER         = credentials('DB_USER')
        DB_PASSWORD     = credentials('DB_PASSWORD')
    }

    triggers {
        // Triggers the pipeline automatically on every GitHub push
        githubPush()
    }

    stages {

        // ─────────────────────────────────────────────
        // 1. Pull latest code from GitHub
        // ─────────────────────────────────────────────
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Devrajj-14/Greeting_App_Development.git'
            }
        }

        // ─────────────────────────────────────────────
        // 2. Build & package the Spring Boot JAR
        // ─────────────────────────────────────────────
        stage('Build') {
            steps {
                // Give execute permission to the Maven wrapper first
                sh 'chmod +x mvnw'
                
                // Then build the application
                sh './mvnw clean package -DskipTests'
            }
        }

        // ─────────────────────────────────────────────
        // 3. Copy JAR to EC2 via SCP
        // ─────────────────────────────────────────────
        stage('Deploy to EC2') {
            steps {
                // 'ec2-ssh-key' is the Jenkins credential ID for your springboot-key.pem
                sshagent(credentials: ['ec2-ssh-key']) {

                    // Copy the JAR
                    sh """
                        scp -o StrictHostKeyChecking=no \
                            target/${JAR_NAME} \
                            ${EC2_USER}@${EC2_HOST}:${DEPLOY_DIR}/
                    """

                    // Stop old instance (if running), then start new one
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            # Kill any previously running instance of the app
                            pkill -f "${JAR_NAME}" || true

                            # Wait a moment for the port to free up
                            sleep 3

                            # Start the app in the background
                            nohup java -jar ${DEPLOY_DIR}/${JAR_NAME} \
                                --server.port=${APP_PORT} \
                                --spring.datasource.url=${DB_URL} \
                                --spring.datasource.username=${DB_USER} \
                                --spring.datasource.password=${DB_PASSWORD} \
                                > ${DEPLOY_DIR}/app.log 2>&1 &

                            echo "App started. Logs at ${DEPLOY_DIR}/app.log"
                        '
                    """
                }
            }
        }
    }

    // ─────────────────────────────────────────────
    // Post-pipeline notifications
    // ─────────────────────────────────────────────
    post {
        success {
            echo "✅ Pipeline succeeded! App deployed to http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above."
        }
    }
}
