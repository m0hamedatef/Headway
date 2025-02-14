pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('access-key')
        AWS_SECRET_ACCESS_KEY = credentials('secret-key')
        ECR_REPOSITORY_URI = '396608774504.dkr.ecr.us-east-1.amazonaws.com'
        AWS_REGION = 'us-east-1'
    }

    stages {
        stage('Configure AWS & Kubernetes') {
            steps {
                script {
                    sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name my_cluster --alias my-cluster
                    export KUBECONFIG=~/.kube/config
                    kubectl get nodes
                    """
                }
            }
        }
        
        stage('Build and Push Images') {
            steps {
                script {
                    sh 'docker build -t ${ECR_REPOSITORY_URI}/graduation:vote-${BUILD_NUMBER} ./app/backend/.'
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY_URI}'
                    sh 'docker push ${ECR_REPOSITORY_URI}/graduation:vote-${BUILD_NUMBER}'

                    sh 'docker build -t ${ECR_REPOSITORY_URI}/graduation:worker-${BUILD_NUMBER} ./app/frontend/.'
                    sh 'docker push ${ECR_REPOSITORY_URI}/graduation:worker-${BUILD_NUMBER}'
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                sed -i 's|image:.*|image: ${ECR_REPOSITORY_URI}/graduation:vote-${BUILD_NUMBER}|g' ./k8s/back.yaml
                sed -i 's|image:.*|image: ${ECR_REPOSITORY_URI}/graduation:worker-${BUILD_NUMBER}|g' ./k8s/front.yaml
                """
            }
        }

        stage('Apply Kubernetes Manifests') {
            steps {
                script {
                    sh """
                    kubectl apply -f ./k8s/ --validate=false
                    """
                }
            }
        }
    }
}
