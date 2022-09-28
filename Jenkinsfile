pipeline {
  agent any
	
	environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "hazemtahri/numeric-app:${GIT_COMMIT}"
    applicationURL="http:devsecops-hazem.eastus.cloudapp.azure.com/"
    applicationURI="/increment/99"
  }

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }  
	  
	  
     stage('Unit Testing') {
            steps {
              sh "mvn test"
              
            }
         
      }
        stage('Mutation Tests - PIT') {
            steps {
               sh "mvn org.pitest:pitest-maven:mutationCoverage"
     }   
		 post {
             always {
              
		pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
		
           }    
        }  
     
          }
	  
	  
    stage('SonarQube SAST') {
            steps {
              withSonarQubeEnv('SonarQube') {
              sh "mvn clean verify sonar:sonar  -Dsonar.projectKey=numeric-app  -Dsonar.host.url=http://devsecops-hazem.eastus.cloudapp.azure.com:9000"  
              
            }
              
                       timeout(time: 2, unit: 'MINUTES') {
           script {
             waitForQualityGate abortPipeline: true
          }
        }
 
     }
    }
  	 stage('Vulnerability Scan - Docker') {
       steps {
         parallel(
         	"Dependency Scan": {
         		sh "mvn dependency-check:check"
			},
			"Trivy Scan":{
	 			sh "bash trivy-docker-image-scan.sh"
	 		},
		 "OPA Conftest":{
				sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
			}
	 		 	
       	)
       }
     }
            
     stage('Docker Build and Push') {
       steps {
         withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
           sh 'printenv'
           sh 'sudo docker build -t hazemtahri/numeric-app:""$GIT_COMMIT"" .'
           sh 'docker push hazemtahri/numeric-app:""$GIT_COMMIT""'
        }
      }
     }
 stage('K8S Deployment - DEV') {
       steps {
         parallel(
           "Deployment": {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash k8s-deployment.sh"
             }
           },
           "Rollout Status": {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash k8s-deployment-rollout-status.sh"
             }
           }
         )
       }
     }
	 post {
             always {
               junit 'target/surefire-reports/*.xml'
               jacoco execPattern: 'target/jacoco.exec'
		
		dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
           }   
        }  
}

}
