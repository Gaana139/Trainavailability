pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gana139/train-schedule"
        DOCKERHUB_CREDENTIALS=credentials('docker_hub_login')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh 'docker version'
                    sh "docker build -t gana139/train-node-app:${BUILD_NUMBER} ."
				    sh 'docker image list'
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
            when {
                branch 'main'
            }
            steps {
                script {
                    //docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        //app.push("${env.BUILD_NUMBER}")
                        //app.push("latest")
                    sh "docker -v"
                        sh "docker push gana139/train-node-app:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'main'
            }
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
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
