pipeline {
    agent any
    tools {
        maven "MAVEN"
        jdk "JDK"
    }
    stages {
        stage('Initialize'){
            steps{
                echo "PATH = ${M2_HOME}/bin:${PATH}"
                echo "M2_HOME = /opt/apache-maven-3.8.2"
                 git 'https://github.com/SWAGATAM04/spring-petclinic.git'
            }
        }
        stage('Build') {
            steps {
                dir("/var/lib/jenkins/workspace/spring-petclinic") {
				sh 'cd spring-petclinic'
                sh './mvnw package'
                sh 'java -jar target/*.jar'
                }
            }
        }
     }
    post {
       always {
          junit(
        allowEmptyResults: true,
        testResults: '*/test-reports/.xml'
      )
      }
   } 
}
