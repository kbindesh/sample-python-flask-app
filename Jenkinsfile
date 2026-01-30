pipeline {
    agent any 
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_creds')
        EKS_CLUSTER_NAME = 'labekscluster'
        AWS_REGION = 'us-east-1'
        AWS_CRED_ID = 'aws-credentials'
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
        stage('Push Docker Image') {
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
        stage('Deploy App to EKS') {
            steps {
                withAWS(credentials: env.AWS_CRED_ID, region: env.AWS_REGION) {
                    sh 'kubectl get pods -A'
                    sh 'kubectl apply -f k8s-specifications\flaskapp-deployment.yml'
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
