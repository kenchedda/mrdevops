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
            
                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "demosnap" : "demowork"

                    nexusArtifactUploader artifacts: 
                    [
                        [
                         artifactId: 'springboot', 
                         classifier: '', file: 'target/Uber.jar',
                         type: 'jar'
                         ]
                         
                    ],
                        credentialsId: 'nexus',
                        groupId: 'com.example',
                        nexusUrl: '3.87.137.55:8081',
                        nexusVersion: 'nexus3',
                        protocol: 'http', 
                        repository: nexusRepo, 
                        version: "${readPomVersion.version}"

                
                }
            }
        }
        stage('Docker image build'){
                    steps {
                        script {
                            sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                            sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kenappiah/$JOB_NAME:latest1'
                        }
                    }
            
                }
            stage ('publish docker image') {
                steps{
                    script{
                        withCredentials([string(credentialsId: 'dockersec', variable: 'docker_hub_cred')]) {
                            sh 'docker login -u kenappiah -p ${docker_hub_cred}'
                            sh 'docker image push kenappiah/$JOB_NAME:latest1'
                    }
                }
            }                
        }
          stage(" Deploy ") {
       steps {
         script {
            echo '<--------------- Helm Deploy Started --------------->'
            sh 'helm install java /home/ec2-user/java-0.1.0.tgz'
            echo '<--------------- Helm deploy Ends --------------->'
         }
       }
     }
    }

    }

    
    
