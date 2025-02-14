pipeline {
    agent any

    environment {
        // Credentials ID from AWS Credentials Plugin
        AWS_ACCESS_KEY_ID = credentials('access-key')
        AWS_SECRET_ACCESS_KEY = credentials('secret-key')
        ECR_REPOSITORY_URI= '796973496394.dkr.ecr.ca-central-1.amazonaws.com'
        AWS_REGION = 'ca-central-1'
    }


    stages {
        
        // stage('Terraform Init and Apply') {
        //     steps {
        //         script {
        //             // Initialize Terraform
        //             sh 'terraform -chdir=terraform init'

        //             // Apply Terraform configurations
        //             sh 'terraform -chdir=terraform apply -auto-approve'
        //         }
        //     }
        // }
        
         stage('Build Docker Image vote') {
             steps {
               
                 sh 'docker build -t ${ECR_REPOSITORY_URI}/headway:vote-${BUILD_NUMBER} ./app/vote/.'
                 sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY_URI}'
                 sh 'docker push ${ECR_REPOSITORY_URI}/headway:vote-${BUILD_NUMBER}'
             }
         }
         stage('Build Docker Image worker') {
             steps {
                 sh 'docker build -t ${ECR_REPOSITORY_URI}/headway:worker-${BUILD_NUMBER} ./app/worker/.'
                 sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY_URI}'
                 sh 'docker push ${ECR_REPOSITORY_URI}/headway:worker-${BUILD_NUMBER}'
             }
         }
         stage('Build Docker Image result') {
             steps {
                 sh 'docker build -t ${ECR_REPOSITORY_URI}/headway:result-${BUILD_NUMBER} ./app/result/.'
                 sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY_URI}'
                 sh 'docker push ${ECR_REPOSITORY_URI}/headway:result-${BUILD_NUMBER}'
             }
         }
        
         stage('Kubernetes Edit Files') {
             steps {
                    sh "sed -i 's|image:.*|image: ${ECR_REPOSITORY_URI}/headway:vote-${BUILD_NUMBER}|g' ./k8s/vote-deployment.yaml"
                    sh "sed -i 's|image:.*|image: ${ECR_REPOSITORY_URI}/headway:worker-${BUILD_NUMBER}|g' ./k8s/worker-deployment.yaml"
                    sh "sed -i 's|image:.*|image: ${ECR_REPOSITORY_URI}/headway:result-${BUILD_NUMBER}|g' ./k8s/result-deployment.yaml"
                      sh "aws eks update-kubeconfig --region ca-central-1 --name master-eks "
             }
        }

        
        stage('apply manifest') {
            steps {

                 sh 'kubectl create -f ./k8s/ '
                 // sh 'kubectl apply -f ./k8s/secret.yaml '  
                 // sh 'kubectl apply -f ./k8s/configmap.yaml '
                 // sh 'kubectl apply -f ./k8s/postgress_serves.yaml '
                 // sh 'kubectl apply -f ./k8s/postgress.yaml '  
            }
        }

        // stage('apply backend') {
        //     steps {
        //           sh 'kubectl apply -f ./k8s/backend.yaml '  
        //           sh 'kubectl apply -f ./k8s/backend_service.yaml '  
        //     }
        // }

        // stage('apply frontend') {
        //     steps {
        //         sh 'kubectl apply -f ./k8s/frontend.yaml '  
        //         sh 'kubectl apply -f ./k8s/frontend_service.yml '  
        //     }
        // }

    }
}
