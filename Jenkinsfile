pipeline {
    agent any
        environment {
        registry = "519852036875.dkr.ecr.us-east-2.amazonaws.com/react-coffee-app"
    }
     stages{
         stage('Build') {
            steps {
                nodejs(nodeJSInstallationName: 'Nodejs'){
                  	 sh'npm cache clean --force'
			sh 'npm install'
		 
                }
           }
        }
         stage("SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('react-sonar') {
                sh 'npm install sonar-scanner'
		            sh'npm i sonar-scanner --save-dev'
		            sh 'npm run sonar-scanner'    	
              }
            }
          }
	     stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
                  }
                 }
	     }
	    
	     
        stage('Building image in EC2') {
      steps{
        script {
            docker.withRegistry( 'https://registry.hub.docker.com/','Jenkins-Docker-Integration'){
             myImage = docker.build ("reactapp:latest")
            }
        }
      }
    }
    stage('Build Registry') {
      steps{
        script {
          dockerImage = docker.build registry
        }
      }
    }
  
     // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 519852036875.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push 519852036875.dkr.ecr.us-east-2.amazonaws.com/react-coffee-app:latest'
         }
        }

      }
	      stage ('K8S Deploy') {
		 steps{
                kubernetesDeploy(
                    configs: 'react_coffee.yml',
                    kubeconfigId: 'Kubernetes',
                    enableConfigSubstitution: true
                    )           
            }
            
		      
        }
      
    }
     
   }
 
