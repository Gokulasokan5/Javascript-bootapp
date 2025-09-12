pipeline {
    agent any 
    tools {
        maven 'maven'
    }
    stages {
        stage('Checkout from Git') { 
            steps {
               git branch: 'prod', url: 'https://github.com/bkrrajmali/morning-javaspringbootapp.git'
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
                    -Dsonar.organization=bkrrajmali \
                    -Dsonar.projectName=petclinic \
                    -Dsonar.projectKey=bkrrajmali_petclinic \
                    -Dsonar.java.binaries=.
                    '''
                } 
            }
        }
        stage('Maven Compile') { 
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
    }
}