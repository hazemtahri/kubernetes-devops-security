pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }  
     stage('unit testing') {
            steps {
              sh "mvn test"
              
            }
         post {
             always {
               junit 'target/surefire-reports/*.xml'
               jacoco-execPatterns: 'target/jacoco.exec'
              
        }   
      }
    }
}
