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
         stage ("Quality Gate") {
            steps {
                script {
                  echo '<--------------- Quality Gate started  --------------->' 
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if(qg.status!='OK'){
                          error "Pipeline failed due to the Quality gate issue"   
                        }    
                    }    
                  echo '<--------------- Quality Gate stopped  --------------->'
                }    
            }   
        } 
        stage('upload war to nexus'){
            steps {
                script {
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo = readMavenPom.version.endsWith("SNAPSHOT") ? "demosnap" : "demowork"
                    nexusArtifactUploader artifacts: 
                    [
                        [artifactId: 'springboot', 
                        classifier: '', file: 'target/Uber.jar',
                         type: 'jar'
                         ]
                         
                         ],
                        credentialsId: 'nexus',
                        groupId: 'com.example',
                        nexusUrl: '18.209.172.31:8081',
                        nexusVersion: 'nexus3',
                        protocol: 'http', 
                        repository: 'nexusRepo', 
                        version: "${readPomVersion.version}"
                }
            }
        }         
        }

    }

    
    
