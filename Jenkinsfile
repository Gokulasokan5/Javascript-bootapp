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
    }
}