pipeline {
    environment {
    registry = "swagatam04/spring-petclinic"
    registryCredential = 'dockerhub'
    AWS_ACCOUNT_ID="098974694488"
    AWS_DEFAULT_REGION="us-east-2" 
    IMAGE_REPO_NAME="demo"
    REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
   
    
  }
    agent any
    stages {
        stage('Initialize'){
            steps{
                echo "PATH = ${M2_HOME}/bin:${PATH}"
                echo "M2_HOME = /opt/apache-maven-3.8.2"
                
            }
        }
        stage('Build') {
            steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                         sh './mvnw package'
                         
                }
            }
        }  
      stage('Sonarqube') {
        
           steps {
               dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                withSonarQubeEnv('sonarqube') {
                      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                      
                }
                 timeout(time: 10, unit: 'MINUTES') {
                              waitForQualityGate abortPipeline: true
        }
      }
    }
  }
        stage('Jfrog Upload') {
             steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD/target") {
                sh 'jfrog c add --artifactory-url="https://swagatamjfrog.jfrog.io/artifactory/" --user="demo" --password="AKCp8mYy3yf14htMsfrogKCdsV9F16Kb7BuLMYSvoBpZPJcR6hWVwjgm1E69Wmb8FKuKQxATP"  --interactive="false"'
                 sh 'jfrog rt u "spring-petclinic-2.6.0-SNAPSHOT.jar" "test/pring-petclinic-2.6.0-SNAPSHOT-$BUILD_NUMBER.jar" --recursive=false'
               
                }
            }
        }  
            
        stage('Test') {
            steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                    sh 'mvn test'
                }
            }
        }
        
     stage('Delete Container and Image') {
      steps{
        sh 'docker ps -a|egrep -i "$registry|$IMAGE_REPO_NAME" |awk \'{print $1}\' |xargs docker rm -f || true'
        sh 'docker images | egrep -i "$registry|$IMAGE_REPO_NAME" | awk \'{print $3}\' |grep -v IMAGE|xargs docker rmi -f || true'
               
      }
    }  
     stage('Building DockerHub image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Push Image to DockerHub') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
        
    stage('Building ECR image') {
      steps{
        script {
          dockerImageAws = docker.build REPOSITORY_URI + ":$BUILD_NUMBER"
        }
      }
    }
     stage('Deploy to AWS ECR') {
            steps {
                script{
                    docker.withRegistry('https://098974694488.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:awscred') {
                    dockerImageAws.push()
                    
                    }
                }
            }
        }
   
    stage('Start Container') {
      steps{
        sh 'docker run -itd -p 8080:8080 -e "SPRING_PROFILES_ACTIVE=postgres"  --link spring-petclinic_petclinic-db.default.svc.cluster.local_1:petclinic-db.default.svc.cluster.local --network spring-petclinic_default $registry:$BUILD_NUMBER'
                
      }
    }
  
   stage('Cleanup Working Directory') {
            steps{
                cleanWs deleteDirs: true
            }
        }
  }
}
