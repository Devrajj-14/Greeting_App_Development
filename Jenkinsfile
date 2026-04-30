pipeline {
    agent any

    environment {
        JAR_NAME        = 'GreetingApp-0.0.1-SNAPSHOT.jar'
        EC2_USER        = 'ubuntu'
        EC2_HOST        = '3.108.40.92'
        DEPLOY_DIR      = '/home/ubuntu'
        APP_PORT        = '9090'
        
        // Hardcode these to match our successful manual test
        DB_URL          = 'jdbc:mysql://localhost:3306/greeting_db?allowPublicKeyRetrieval=true'
        DB_USER         = 'devraj'
        
        // Keep pulling the password securely
        DB_PASSWORD     = credentials('DB_PASSWORD') 
    }

    triggers {
        // Automatically starts build when code is pushed to GitHub
        githubPush()
    }

    stages {
        // 1. Pull code from the master branch
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Devrajj-14/Greeting_App_Development.git'
            }
        }

        // 2. Build the JAR using Maven (Skip tests for speed)
        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        // 3. Securely deploy to the new EC2 instance
        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    // Copy the fresh JAR to the server
                    sh "scp -o StrictHostKeyChecking=no target/${JAR_NAME} ${EC2_USER}@${EC2_HOST}:${DEPLOY_DIR}/"

                    // Start the application remotely as a single string block
                    // Using << EOF (without single quotes) so Jenkins fills in the variables
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            # Stop any old version of the app
                            pkill -f ${JAR_NAME} || true
                            sleep 2

                            # Launch the app with nohup so it stays running after Jenkins disconnects
                            # Secrets are wrapped in single quotes to protect special characters
                            nohup java -jar ${DEPLOY_DIR}/${JAR_NAME} \
                                --server.port=${APP_PORT} \
                                --spring.datasource.url='${DB_URL}' \
                                --spring.datasource.username='${DB_USER}' \
                                --spring.datasource.password='${DB_PASSWORD}' \
                                > ${DEPLOY_DIR}/app.log 2>&1 &
                            
                            # Give the process a moment to stabilize before closing the tunnel
                            sleep 5
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: App deployed to http://${EC2_HOST}:${APP_PORT}/greeting"
        }
        failure {
            echo "❌ FAILURE: Pipeline failed. Check 'Console Output' for details."
        }
    }
}
