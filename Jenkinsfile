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
                
            }
        }
        stage('Build') {
            steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                sh './mvnw package'
               
                }
            }
        }
        stage('Deploy') {
            steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                    sh 'if [ $(netstat -tulnp|grep 8080|wc -l) -eq 0 ]; then echo "No process is running"; else kill -9 $(netstat -tulnp|grep 8080|awk "{print $7}"|tr -d "/java"); fi'
'
                    sh 'echo starting petclinic app'
                    sh 'nohup java -jar target/*.jar &'
                }
            }
        }
     }
   
}
