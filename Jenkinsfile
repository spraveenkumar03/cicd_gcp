pipeline {
    agent any

    environment {
        PROJECT_ID = "cicd-27feb"
        CLUSTER_NAME = "my-gke-cluster"
        CLUSTER_ZONE = "asia-south1-a"
        REPO_NAME = "cicd"
        IMAGE_NAME = "my-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        REGION = "asia-south1"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // stage('Authenticate to GCP') {
        //     steps {
        //         withCredentials([file(credentialsId: 'gcp-key', variable: 'GCP_KEY')]) {
        //             sh '''
        //             gcloud auth activate-service-account --key-file=$GCP_KEY
        //             gcloud config set project $PROJECT_ID
        //             '''
        //         }
        //     }
        // }

        stage('Build Image') {
            steps {
                sh '''
                export PATH=/usr/local/bin:$PATH
                docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                export PATH=/Users/praveen/google-cloud-sdk/bin:/usr/local/bin:$PATH
                echo "PATH is: $PATH"
                which gcloud
                which docker
                gcloud --version
                gcloud config set project cicd-27feb
                gcloud auth configure-docker $REGION-docker.pkg.dev
                docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_ZONE

                kubectl set image deployment/gke-cicd-demo \
                gke-cicd-demo=$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG \
                --record

                kubectl rollout status deployment/gke-cicd-demo
                '''
            }
        }
    }
}