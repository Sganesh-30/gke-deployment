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
                        gcloud components install gke-gcloud-auth-plugin
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
                    docker push asia-south1-docker.pkg/$PROJECT_ID/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
        stage('Deploy to GKE'){
            steps {
                sh '''
                    kubectl set image deployment/gke-app gke-app=asia-south1-docker.pkg.dev/$PROJECT_ID/$IMAGE_NAME:$IMAGE_TAG --record
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