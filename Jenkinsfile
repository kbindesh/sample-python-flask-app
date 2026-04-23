pipeline {
    agent any 
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-creds')
        AWS_CRED_ID = 'aws-credentials'
        EKS_CLUSTER_NAME = 'labekscluster'
        AWS_REGION = 'us-east-1'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        PATH = "$PATH:/opt"
    }
    stages { 

        stage('Building an Image') {
            steps {  
                sh 'docker build -t kbindesh/flaskapp:$BUILD_NUMBER .'
            }
        }
        stage('Connecting to DockerHub') {
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                echo "Successfully connected to DockerHub Account"
            }
        }
        stage('Push Image to Registry') {
            steps{
                sh 'docker push kbindesh/flaskapp:$BUILD_NUMBER'
                echo "Pushed the kbindesh/flaskapp:${BUILD_NUMBER} image successfully"
            }
        }
        stage('Configure EKS kubeconfig') {
            steps {
                // Use the AWS CLI to authenticate and update kubeconfig
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AWS_CRED_ID]]) {
                    sh "aws configure set region ${AWS_REGION}"
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION}"
                    echo "Configured kubeconfig for EKS cluster: ${EKS_CLUSTER_NAME}"
                }
            }
        }
        stage('Update Image Tag') {
            steps {
                // Replace placeholder with actual tag
                sh "sed -i 's|__IMAGE_TAG__|${IMAGE_TAG}|g' k8s-specifications/flaskapp-deployment.yml"
            }
        }
        stage('Deploy App to EKS Cluster') {
            steps {
                withAWS(credentials: env.AWS_CRED_ID, region: env.AWS_REGION) {
                    sh 'kubectl get pods -A'
                    sh 'kubectl apply -f k8s-specifications/flaskapp-deployment.yml'
                    echo "Python Flask app deployed successfully"
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
