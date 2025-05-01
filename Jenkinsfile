pipeline {
    agent any
        environment {
        AWS_REGION = 'us-east-1' 
        EKS_CLUSTER_NAME = 'streamlit-cluster'
        DEPLOYMENT_NAME = 'streamlit' 
        K8S_MANIFESTS_DIR = 'kubernetes' 
    }
        stages {
            stage('Clean Workspace') {
                steps {
                    cleanWs()
                }
            }
            
            stage('Checkout') {
                steps {
                    git 'https://github.com/mv2727/streamlit-app.git'
                }
            }
            
            stage('Image building using Docker') {
                steps {
                    sh 'docker build -t vm27/streamlit-app .'
                }
            }
            
            stage('Docker Push'){
                steps{
                    script{
                        withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Use the credentials to log in to Docker
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                        sh 'docker push vm27/streamlitapp:latest'
                        
                    }
                }
            }
           
      }
      
      stage('Deploying to EKS cluster'){
          steps {
              script{
                  sh 'aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER_NAME'
                  sh "kubectl apply -f ${K8S_MANIFESTS_DIR}/deployment.yaml"
                  sh "kubectl apply -f ${K8S_MANIFESTS_DIR}/service.yaml"
              }
              }
          }


      }
      
       post {
        success {
            echo 'Deployment to EKS was successful!'
        }
        failure {
            echo 'Deployment to EKS failed.'
        }
      
    }
    
}

