/* groovylint-disable CompileStatic, DuplicateStringLiteral, NestedBlockDepth */
pipeline {
    agent any

    stages {

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_REGISTRY = 'neroxxpips'
                DOCKER_IMAGE_NAME = 'oe-final-test'
                DOCKER_IMAGE_TAG = 'latest'
                DOCKER_PASSWORD = credentials('docker-token')
                DOCKER_USERNAME = 'neroxxpips'
            }
            steps {
                sh 'docker build -t $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG .'
                sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                sh 'docker push $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG'
                sh 'docker logout'
            }
        }

        stage('Deploy Application in to K8s') {
            environment {
                EKS_CLUSTER_NAME = 'OE-DevOps-cluster'
                AWS_REGION = 'us-east-1'
                NAMESPACE = 'oe-jenkins'
                IMAGE_NAME = 'oe-final-test'
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                    script {
                        sh('aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION')
                        sh 'sed -i "s/__IMAGE_NAME__/$IMAGE_NAME/" deploy/deployment.yaml'
                        sh 'sed -i "s/__NAMESPACE__/$NAMESPACE/" deploy/namespace.yaml'
                        sh 'kubectl apply -f deploy/namespace.yaml'
                        sh 'kubectl apply -f deploy/deployment.yaml -f deploy/service.yaml --namespace $NAMESPACE'
                    }
                }
            }
        }
    }
}
