pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "<your-dockerhub-username>/nodejs-app:latest"
        AWS_DEFAULT_REGION = "us-west-2"
        CLUSTER_NAME = "example-eks-cluster"
        HELM_CHART_DIR = "helm-chart"
        APP_NAME = "nodejs-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('nodejs-app') {
                    script {
                        docker.build(env.DOCKER_IMAGE)
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    script {
                        docker.withRegistry('', 'dockerhub-creds') {
                            docker.image(env.DOCKER_IMAGE).push()
                        }
                    }
                }
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $CLUSTER_NAME
                    """
                }
            }
        }

        stage('Deploy to EKS with Helm') {
            steps {
                sh """
                    helm upgrade --install $APP_NAME $HELM_CHART_DIR \
                        --set image.repository=<your-dockerhub-username>/nodejs-app \
                        --set image.tag=latest
                """
            }
        }
    }
}
