pipeline {
    agent any 
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_creds')
        EKS_CLUSTER_NAME = 'labekscluster'
        EKS_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = ''
        AWS_SECRET_KEY_ID = ''
    }
    stages { 

        stage('Build image') {
            steps {  
                sh 'docker build -t kbindesh/flaskapp:$BUILD_NUMBER .'
            }
        }
        stage('Test image'){
            steps {
                 echo 'Empty'
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
        stage('K8s Deploy') {
            steps {
                script {
                    withAWS(credentials: 'AWS_CREDS', region: $EKS_REGION) {
                        sh 'kubectl version'
                        sh 'aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $EKS_REGION'
                        sh 'kubectl apply -f deployment.yaml'
                    }
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
