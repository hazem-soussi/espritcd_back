pipeline {
agent any
    
    
        environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.149.136:8081"
        NEXUS_REPOSITORY = "achat"
        NEXUS_CREDENTIAL_ID = "nexus_auth"
    }
    
    stages {
        

        // main SATGES
        stage ("1st stage : Git checkout PLEAASE"){
            steps{
        git branch: 'main', 
            url: 'https://github.com/hazem-soussi/espritcd_back.git'
            }
        
        }
    
        
                stage('2nd Stage : Maven Build Project') {
            steps {
                echo "Build our project"
                sh 'mvn clean install '
            }
        }
        
       
             stage ("3rd Stage : unit testing"){
            steps{
                sh "mvn test"
            }
        
        }
        
        stage ("4th Stage : Integration testing"){
            
                steps {
                sh "mvn verify -DskipUnitTests"
                
                }
            }
        
        stage ("5th Stage : SONARQUBE Analysis"){
            steps {
            echo 'Analzying quality code.'
                script {
                    withSonarQubeEnv(credentialsId: 'hazem_sonar_ci', installationName: 'sonarqube_server') {
     
                  sh "mvn clean package sonar:sonar" 

                 }
                }
              }
            
        }
     
        // ********************************** 
        
       stage ('Quality Gate Status'){
            steps {
                script{
                    withSonarQubeEnv(credentialsId: 'hazem_sonar_ci', installationName: 'sonarqube_server') {

                    }
                
                
                }
            
            }
        }
        
           stage("6th Stage : Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        
        stage ("7th stage :Docker image build") {
            steps {
                script {
                sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                sh 'docker image tag $JOB_NAME:v1.$BUILD_ID hazem1998/$JOB_NAME:v1.$BUILD_ID '
                sh 'docker image tag $JOB_NAME:v1.$BUILD_ID hazem1998/$JOB_NAME:latest '

                }
        
        
        }
        }
        
        
        stage ("Push image to dockerHUb") {
        
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_password', variable: 'docker_imagePWD')]) {
                        sh "docker login -u hazem1998 -p ${docker_imagePWD}"
                    sh 'docker image push hazem1998/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image push hazem1998/$JOB_NAME:latest '}       
                }
            
            }
        
        
        }
        
        
         
        stage("APP CONTAINERS LAUNCH WITH Docker-Compose") {
          steps {
              sh 'docker-compose up'
             }
     
       } 
    
        
             //stage("6th stage : packaging & deploy stage ") {
    //   steps
        // {
           // sh "mvn clean -DskipTests package deploy:deploy-file -DgroupId=com.esprit.examen -DartifactId=tpAchatProject -Dversion=1.0.0 -DgeneratePom=true -Dpackaging=war -DrepositoryId=esprit-app -Durl=http://192.168.149.136:8081/repository/maven-releases/ -Dfile=target/tpAchatProject-1.0.0.war"
      //  }
//} 
        
            

                        
                        

    
    
    }
    
    
  
}
