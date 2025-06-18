pipeline {
    agent any 

    environment {
        PROJECT_ID = 'dotted-task-461114-m5'
        IMAGE_NAME = 'gke-app/gke-sampleapp'
        IMAGE_TAG = 'v1'
        REGION = 'asia-south1'
        CLUSTER = 'sampleapp-cluster'
        CLUSTER_ZONE = 'asia-south1-a'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sganesh-30/gke-deployment.git'
            }
        }

        stage('Auth GCloud') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project $PROJECT_ID
                        gcloud config set compute/zone $CLUSTER_ZONE
                        gcloud container clusters get-credentials $CLUSTER --zone $CLUSTER_ZONE
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t asia-south1-docker.pkg.dev/$PROJECT_ID/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }
        stage('Push to GCR') {
            steps {
                sh '''
                    gcloud auth configure-docker $REGION-docker.pkg.dev
                    docker push asia-south1-docker.pkg.dev/$PROJECT_ID/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
        stage('Updating image tag'){
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Sganesh-30/gke-deployment.git'
                    cd Kubernetes
                    sh '''
                        sed -i 's|image: asia-south1-docker.pkg.dev${IMAGE_NAME}:.*|image: asia-south1-docker.pkg.dev${IMAGE_NAME}:${BUILD_TAG}|' /Kubernetes/deployment.yaml
                    '''                 
                }
            }
        }
        stage('Deploy to GKE'){
            steps {
                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}