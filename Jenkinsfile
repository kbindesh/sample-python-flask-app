pipeline {
    agent any 
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_creds')
        EKS_CLUSTER_NAME = 'labekscluster'
        AWS_REGION = 'us-east-1'
        AWS_CRED_ID = 'aws-credentials'
    }
    stages { 

        stage('Build image') {
            steps {  
                sh 'docker build -t kbindesh/flaskapp:$BUILD_NUMBER .'
            }
        }
        stage('Connecting to DockerHub') {
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push image') {
            steps{
                sh 'docker push kbindesh/flaskapp:$BUILD_NUMBER'
            }
        }
        stage('Configure AWS Credentials and EKS Access') {
            steps {
                // Use the AWS CLI to authenticate and update kubeconfig
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AWS_CRED_ID]]) {
                    sh "aws configure set region ${AWS_REGION}"
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION}"
                    echo "Configured kubeconfig for EKS cluster: ${EKS_CLUSTER_NAME}"
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                // Use the ID defined in the Jenkins Credentials Manager
                withAWS(credentials: env.AWS_CRED_ID, region: env.AWS_REGION) { 
                    sh 'aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION}'
                    sh '/opt/kubectl get nodes' // Example kubectl command
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
