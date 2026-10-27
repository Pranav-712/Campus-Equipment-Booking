
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Package JAR') {
            steps {
                sh 'mvn package -DskipTests'
                sh 'test -f target/campus-equipment-booking-1.0.0.jar'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t campus-equipment-booking:set-d .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f campus-equipment-booking-container || true
                    docker run -d --name campus-equipment-booking-container \
                        -p 8082:8080 campus-equipment-booking:set-d
                '''
            }
        }

        stage('Verify Health') {
            steps {
                sh '''
                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:8082/actuator/health; then
                            echo
                            exit 0
                        fi
                        sleep 2
                    done
                    echo "Health check failed"
                    docker logs campus-equipment-booking-container
                    exit 1
                '''
            }
        }
    }
}
