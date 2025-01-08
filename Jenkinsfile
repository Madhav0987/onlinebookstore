pipeline {
    agent any
    environment {
        REPOSITORY_URI = "515966525347.dkr.ecr.ap-south-1.amazonaws.com/assignment"
		SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Madhav0987/docker-dec-22.git'
            }
        }
		stage('building artifact and unit testing'){
		    steps{
			    sh "mvn install package"
				}
			}
		stage('Sonarqube analysis'){
	     steps{
	         withSonarQubeEnv('Sonar-server') {
	             sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://15.207.107.47:9000/ -Dsonar.projectName="Java WebApp" \
	              -Dsonar.java.binaries=. \
                  -Dsonar.projectKey=Java-WebApp '''

        }
       }
      }
        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ubuntu:v1 .
                """
            }
        }
        stage('Push to AWS ECR') {
            steps {
                script {
                    withEnv(["AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}", "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}", "AWS_DEFAULT_REGION=${env.AWS_DEFAULT_REGION}"])
{
                    sh """
                        aws ecr get-login-password --region ap-south-1 | \
                        docker login --username AWS --password-stdin 515966525347.dkr.ecr.ap-south-1.amazonaws.com
                        docker tag ubuntu:v1 ${REPOSITORY_URI}:v1
                        docker push ${REPOSITORY_URI}:v1
                    """
                }
}
            }
        }

        stage('deplyoing app to kubernetes){
		    steps{
			  withKubeConfig([credentialsId: 'kubeconfig-secret-id']) {
                    sh 'kubectl apply -f deployment.yaml'
                    
                }
            }
        }
			   
    }
}
