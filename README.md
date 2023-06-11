# Spring Boot based Java web application
 
This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.

## Execute the application locally and access it using your browser

Checkout the repo and move to the directory

```
git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/sprint-boot-app
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app
```

Execute the Maven targets to generate the artifacts

```
mvn clean package
```

The above maven target stroes the artifacts to the `target` directory. You can either execute the artifact on your local machine
(or) run it as a Docker container.

** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **


### Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```

### The Docker way

Build the Docker Image

```
docker build -t ultimate-cicd-pipeline:v1 .
```

```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Hurray !! Access the application on `http://<ip-address>:8010`


## Next Steps

### Configure a Sonar Server locally

```
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 

## Next Steps for deployment
below is the jenkins pipeline added.
for deploying in in kubernetes, use helm charts
update the docker repo, and docker hub credentials, and git login
update the image name from the values.yaml 

```
# update the docker  repo  <docker repo>, 
# add dockerhub_login credentials  in jenkins
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
```


