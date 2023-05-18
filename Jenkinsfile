pipeline {
    agent {
        node{
            label "build"
        }
    }
    environment {
        PATH= "/opt/apache-maven-3.9.2/bin:$PATH"
      }   
    stages {
        
        stage('Git Checkout'){
            steps{
                git branch: 'main', credentialsId: 'git', url: 'https://github.com/kenchedda/mrdevops.git'
            }
        }
        stage( 'Unit Test'){
            steps{
                sh 'mvn test'

            }
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
       stage ("Sonar Analysis") {
            environment {
               scannerHome = tool 'sonar'  //scanner name configured for slave 
            }
            steps {
                echo '<--------------- Sonar Analysis started  --------------->'
                withSonarQubeEnv('sonar') {    
                    //sonarqube server name in master
                    sh "${scannerHome}/bin/sonar-scanner"
                }    
                echo '<--------------- Sonar Analysis stopped  --------------->'
            }   
        }
        stage('quality gate'){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'mrdevops'
            }

        }    
        }

    }

    
    
