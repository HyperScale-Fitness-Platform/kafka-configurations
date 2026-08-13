pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: aws-k8s
                # Alpine-based image containing aws-cli, kubectl, and helm
                image: alpine/k8s:1.30.0 
                command:
                - cat
                tty: true
            '''
        }
    }

    environment {
        KUBERNETES_DIR = "${WORKSPACE}/k8s"
        NAMESPACE      = "gym-dev"
        AWS_REGION     = "us-east-1"
        CLUSTER_NAME   = "gym-cluster"
        
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Initialize EKS Context') {
            steps {
                // Execute commands explicitly inside the tool container
                container('aws-k8s') {
                    echo '🛡️ Updating cluster context connection...'
                    sh "aws eks update-kubeconfig --region ${env.AWS_REGION} --name ${env.CLUSTER_NAME}"
                }
            }
        }

        stage('Deploy Kafka Broker') {
            steps {
                container('aws-k8s') {
                    echo '📦 Deploying Apache Kafka (KRaft) to Kubernetes...'
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/configmap.yaml"
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/persistentvolumeclaim.yaml"
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/deployment.yaml"
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/service.yaml"

                    echo '⏳ Waiting for Kafka broker rollout...'
                    sh "kubectl rollout status deployment/kafka -n ${env.NAMESPACE} --timeout=180s"
                }
            }
        }

        stage('Deploy Kafka UI') {
            steps {
                container('aws-k8s') {
                    echo '📊 Deploying Kafka UI...'
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/kafka-ui/deployment.yaml"
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/kafka-ui/service.yaml"
                    sh "kubectl apply -f ${env.KUBERNETES_DIR}/kafka-ui/ingress.yaml"

                    echo '⏳ Waiting for Kafka UI rollout...'
                    sh "kubectl rollout status deployment/kafka-ui -n ${env.NAMESPACE} --timeout=120s"
                }
            }
        }

        stage('Smoke Test') {
            steps {
                container('aws-k8s') {
                    echo '🧪 Verifying Kafka broker is reachable...'
                    sh "kubectl run kafka-smoke --rm -i --restart=Never -n ${env.NAMESPACE} --image=busybox:1.35.0 -- sh -c 'nc -z kafka-service 9092 && echo KAFKA_UP'"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Kafka + Kafka UI successfully deployed to ${env.CLUSTER_NAME}!"
        }
        failure {
            echo "❌ Kafka deployment failed! Check the step diagnostics above."
        }
    }
}