pipeline {
    agent any

    environment {
        JAR_NAME        = 'GreetingApp-0.0.1-SNAPSHOT.jar'
        EC2_USER        = 'ubuntu'
        EC2_HOST        = '3.108.40.92'
        DEPLOY_DIR      = '/home/ubuntu'
        APP_PORT        = '9090'
        DB_URL          = credentials('DB_URL')
        DB_USER         = credentials('DB_USER')
        DB_PASSWORD     = credentials('DB_PASSWORD')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Devrajj-14/Greeting_App_Development.git'
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    // Upload the JAR
                    sh "scp -o StrictHostKeyChecking=no target/${JAR_NAME} ${EC2_USER}@${EC2_HOST}:${DEPLOY_DIR}/"

                    // Start the app using Here-Doc (EOF) for maximum stability
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
                            # Kill old process if it exists
                            pgrep -f ${JAR_NAME} | xargs kill -9 || true
                            
                            # Navigate to home
                            cd ${DEPLOY_DIR}

                            # Launch with full paths and single quotes for DB credentials
                            nohup java -jar ${JAR_NAME} \
                                --server.port=${APP_PORT} \
                                --spring.datasource.url='${DB_URL}' \
                                --spring.datasource.username='${DB_USER}' \
                                --spring.datasource.password='${DB_PASSWORD}' \
                                > app.log 2>&1 &
                            
                            # Important: small sleep to let the shell detach cleanly
                            sleep 5
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: App is deploying! Check http://${EC2_HOST}:${APP_PORT}/greeting in 30 seconds."
        }
        failure {
            echo "❌ FAILURE: Pipeline failed. Check the Jenkins Console Output."
        }
    }
}
