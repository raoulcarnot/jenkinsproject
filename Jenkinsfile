pipeline {
    tools {
        maven 'maven'
    }
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/raoulcarnot/jenkinsproject.git']]])
            }
        }
        stage('Build Jar') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Image Build') {
            steps {
                sh 'docker build -t devoprojet .'
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'awscredentials', region: 'eu-central-1') {
                    sh 'aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 150741279371.dkr.ecr.eu-central-1.amazonaws.com'
                    sh 'docker tag devoprojet:latest 150741279371.dkr.ecr.eu-central-1.amazonaws.com/devoprojet:latest'
                    sh 'docker push 150741279371.dkr.ecr.eu-central-1.amazonaws.com/devoprojet:latest'
                }
            }
        }
        stage('Integrate Jenkins with EKS Cluster and Deploy App') {
            steps {
                withAWS(credentials: 'awscredentials', region: 'eu-central-1') {
                  script {
                    sh ('aws eks --region eu-central-1 update-kubeconfig --name raoul1cluster')
                    sh '/var/lib/jenkins/kubectl apply -f eks-deploy-k8s.yaml'
                }
                }
        }
    }
    }
}
