pipeline {
    agent any

    environment {
        JAR_NAME        = 'GreetingApp-0.0.1-SNAPSHOT.jar'
        EC2_USER        = 'ubuntu'
        EC2_HOST        = '3.108.40.92'
        DEPLOY_DIR      = '/home/ubuntu'
        APP_PORT        = '9090'
        // Accessing credentials correctly for shell usage
        DB_URL          = credentials('DB_URL')
        DB_USER         = credentials('DB_USER')
        DB_PASSWORD     = credentials('DB_PASSWORD')
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Devrajj-14/Greeting_App_Development.git'
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
                    // 1. Copy the JAR file to the server
                    sh "scp -o StrictHostKeyChecking=no target/${JAR_NAME} ${EC2_USER}@${EC2_HOST}:${DEPLOY_DIR}/"

                    // 2. Execute the startup sequence as a single quoted string to prevent connection drops
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                            pkill -f ${JAR_NAME} || true
                            sleep 2
                            nohup java -jar ${DEPLOY_DIR}/${JAR_NAME} \
                                --server.port=${APP_PORT} \
                                --spring.datasource.url='${DB_URL}' \
                                --spring.datasource.username='${DB_USER}' \
                                --spring.datasource.password='${DB_PASSWORD}' \
                                > ${DEPLOY_DIR}/app.log 2>&1 &
                            sleep 2
                            exit
                        "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded! App deployed to http://${EC2_HOST}:${APP_PORT}/greeting"
        }
        failure {
            echo "❌ Pipeline failed. Check the Console Output for errors."
        }
    }
}
