pipeline {
    agent any
    tools {
        jdk "jdk11"
        maven "maven3"
    }

    environment {
        SCANNER_HOME = tool 'sonar'
        DOCKER_IMAGE_NAME = "tomcat"
        AWS_REGION = 'us-east-1'  
        ECR_REPOSITORY = 'demo'  
        IMAGE_TAG = "latest"  
        AWS_ACCOUNT_ID = '481665090399'  
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}"
        EKS_CLUSTER_NAME = 'OPQ'  
        KUBECONFIG = "${HOME}/.kube/config"  

    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Chandan-080196/java-onlinebookstore.git'
            }
        }

        stage('Clean and Install') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=java-onlinebookstore \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=java-onlinebookstore'''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    
                    sh """
                        docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} .
                        docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                 script {
                     withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS cli']]) {
                      sh """
                           aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                      """
                 }
    	    }
	}
    }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                          sh "docker push ${ECR_URI}:${IMAGE_TAG}"
                }
            }
        }
   stage('Update Kubeconfig') {
    steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS cli']]) {
            script {
                env.AWS_REGION = 'us-east-1'
                env.EKS_CLUSTER_NAME = 'OPQ'

                sh '''
                    aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                '''
            }
        }
    }
}
 }
}
