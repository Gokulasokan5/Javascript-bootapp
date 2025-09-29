pipeline {
    agent any 
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME  = "sprinbootapp"
        IMAGE_TAG  = "latest"
        ACR_NAME    = "ncpldocker55"
        ACR_LOGIN_SERVER = "ncpldocker55.azurecr.io"
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        TENANT_ID   = "fcdb9b0c-aa0e-4526-857b-3cf898568a89"
        RESOURCE_GROUP = "demo-rg"
        CLUSTER_NAME    = "demo-eks"
    }
    stages {
        stage('Checkout from Git') { 
            steps {
               git branch: 'prod', url: 'https://github.com/Gokulasokan5/Javascript-bootapp.git'
            }
        }
        stage('Maven Validate') { 
            steps {
               sh 'mvn validate'
            }
        }
        stage('Maven Compile') { 
            steps {
               sh 'mvn compile'
            }
        }
        stage('Sonar Analysis') { 
            environment {
                SCANNER_HOME = tool 'Sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarserver'){
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.organization=gokul55 \
                    -Dsonar.projectName=Petclinic \
                    -Dsonar.projectKey=gokul55_petclinic \
                    -Dsonar.java.binaries=.
                    '''
                } 
            }
        }
    
        stage('Maven Package') { 
            steps {
               sh 'mvn package'
            }
        }
        stage('Sonar Quality Gate') {
         steps {
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
            }
         }
       }
       stage('Docker Build'){
        steps {
            script {
                echo "Docker Image Build"
                docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
            }
        }
       }
       stage ('Azure Login to ACR') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD' )]) {
               script {
                echo "Login to Azure" 
                sh '''
                az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                az acr login --name $ACR_NAME       
                '''
                }
            }
         }
       }
       stage ('Docker Push to ACR') {
        steps {
            script {
                echo "Docker Image Push"
                sh '''
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
                docker push ${FULL_IMAGE_NAME}
                '''
            }
         }
       }
       stage ('Login to AKS Cluster') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD' )]) {
            script {
                echo "Azure Login to AKS"
                sh '''
                az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                az aks get-credentials --resource-group $RESOURCE_GROUP   --name $CLUSTER_NAME --overwrite-existing
                '''
             }
            }
          }
       }
       stage ('Deploy to AKS') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD' )]) {
            script {
                echo "Azure Login to AKS"
                sh '''
                az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                kubectl apply -f k8s/sprinboot-deployment.yaml
                '''
             }
            }
          }
       }
    }
}