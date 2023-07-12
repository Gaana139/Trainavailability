pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gana139/train-schedule"
        DOCKERHUB_CREDENTIALS=credentials('docker_hub_login')
    }
    stages {
        stage('SCM Checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/Gaana139/Train_Pipeline.git'
            }
		}
        stage('Build') {
            steps {
                echo 'Running build automation'
                //sh './gradlew build --no-daemon'
                //archiveArtifacts artifacts: 'dist/trainSchedule.zip'
                //sh 'gradle -v'
                //archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            
            steps {
                script {
                    sh 'docker version'
                    sh "docker build -t gana139/train-node-app:${BUILD_NUMBER} ."
				    sh 'docker image list'
				    //sh "docker tag gana139/train-node-app:${BUILD_NUMBER} gana139/train-node-app:latest"
                    //app = docker.build(DOCKER_IMAGE_NAME)
                    //app.inside {
                        //sh 'echo Hello, World!'
                    //}
                }
            }
        }  
        stage('Login2DockerHub') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
        stage('Push Docker Image') {
            
            steps {
                script {
                    //docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        //app.push("${env.BUILD_NUMBER}")
                        //app.push("latest")
                        sh "docker -v"
                        sh "docker push gana139/train-node-app:${BUILD_NUMBER}"
                        
                    //}
                }
            }
        }
       stage('CanaryDeploy') {
            
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        } 
    }
    
}
