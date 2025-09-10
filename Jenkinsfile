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
    }
}