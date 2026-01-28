pipeline {
    agent any 
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_creds')
        EKS_CLUSTER_NAME = 'labekscluster'
        EKS_REGION = 'us-east-1'
        AWS_CRED_ID = 'AWS_CREDS'
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
        stage('Deploy to EKS') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://C032F974A18F6C2A8C3B3F2F1BDC08DD.gr7.us-east-1.eks.amazonaws.com']) {
                        sh 'kubectl version'
                        sh 'kubectl get nodes'
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
