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
        }   
    }
}
