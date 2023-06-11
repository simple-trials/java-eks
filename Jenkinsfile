# update the docker  repo  <docker repo>, dockerhub_login in jenkins
pipeline {
    agent any
	environment {
		DOCKERHUB_CREDENTIALS = credentials('dockerhub_login')
	}
    stages {      
       
        stage('Checkout') {
            agent {label 'Docker'}
			steps {
				git branch: 'main',  url: 'https://github.com/simple-trials/java-eks.git'
            }
		}
   	
        stage('Build') {
            agent {label 'Docker'}
            steps {
				sh "sudo mvn clean install"
				sh "docker build --tag <docker repo>:${JOB_NAME}_$BUILD_NUMBER ."
			}	
        }	
		stage('Docker Push') { 
		agent{label 'Docker'}
			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
				sh "docker push  <docker repo>:${JOB_NAME}_$BUILD_NUMBER"
			} 
        }
		stage('DEPLOY in k8s') {
		agent{label 'eks'}
				steps {
				    git branch: 'main', credentialsId: 'git-login', url: 'https://github.com/simple-trials/java-eks.git'
					sh 'helm upgrade --install java-app helm/ --set image.repository= <docker repo>:${JOB_NAME}_$BUILD_NUMBER'
			}
		} 
	}
 }
