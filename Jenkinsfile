
pipeline {
    agent any

    environment {
        K8S_SERVER = 'https://192.168.58.2:8443'
        K8S_CA = '/etc/jenkins/kubernetes/ca.crt'
        K8S_NAMESPACE = 'default'
    }

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
                sh '''
                    IMAGE="campus-equipment-booking:build-${BUILD_NUMBER}"
                    docker build -t "$IMAGE" .
                '''
            }
        }

        stage('Load Image into Minikube') {
            steps {
                sh '''
                    set -eu
                    IMAGE="campus-equipment-booking:build-${BUILD_NUMBER}"

                    docker save "$IMAGE" |
                        docker exec -i minikube ctr -n k8s.io images import -

                    docker exec minikube ctr -n k8s.io images ls |
                        grep -F "$IMAGE"
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'k8s-deployer-token',
                        variable: 'K8S_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu
                        IMAGE="campus-equipment-booking:build-${BUILD_NUMBER}"

                        kubectl \
                            --server="$K8S_SERVER" \
                            --certificate-authority="$K8S_CA" \
                            --token="$K8S_TOKEN" \
                            --namespace="$K8S_NAMESPACE" \
                            set image \
                            deployment/campus-equipment-booking \
                            campus-equipment-booking="$IMAGE"

                        kubectl \
                            --server="$K8S_SERVER" \
                            --certificate-authority="$K8S_CA" \
                            --token="$K8S_TOKEN" \
                            --namespace="$K8S_NAMESPACE" \
                            rollout status \
                            deployment/campus-equipment-booking \
                            --timeout=180s
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'k8s-deployer-token',
                        variable: 'K8S_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu

                        kubectl \
                            --server="$K8S_SERVER" \
                            --certificate-authority="$K8S_CA" \
                            --token="$K8S_TOKEN" \
                            --namespace="$K8S_NAMESPACE" \
                            get deployment campus-equipment-booking

                        kubectl \
                            --server="$K8S_SERVER" \
                            --certificate-authority="$K8S_CA" \
                            --token="$K8S_TOKEN" \
                            --namespace="$K8S_NAMESPACE" \
                            get pods -l app=campus-equipment-booking
                    '''
                }
            }
        }
    }
}
