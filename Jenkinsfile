pipeline {
    agent {
        node{
            label "build"
        }
    }
    stages {
        
        stage('Git Checkout'){
            steps{
                git branch: 'main', credentialsId: 'git', url: 'https://github.com/kenchedda/mrdevops.git'
            }
        stage( 'Unit Test'){
            steps{
                sh 'mvn test'

            }
        stage('integration test'){
            steps{
                sh 'mvn verify -Dmaven.test.skip=true'
            }
        }
        stage('build'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis'){
            steps{
            withSonarQubeEnv(credentialsId: 'mrdevops') {
                        
                sh 'mvn clean package sonar:sonar'
            }
        }    
        }
        stage('quality gate'){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'mrdevops'
            }

        }    
        }

    }

    
}