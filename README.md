# Docker-PageObjectModel_Selenium

This is a repositry that contains logic to get started with Selenium test auomtaion architecture following page object model design pattern. This readme file will help you provide idea on how to setup Jenkins to publish changes to architecture to the Docker hub when there are changes posted in gitlab and also to run tests in Jenkins. 

After cloning the project. Here are the steps to ensure that workspace is setup correctly.

1. run `mvn clean package -DSkipTests`. This will ensure that automation project and the dependencies are available as Jars under target folder.
2. Have selenium grid setup in a different location using a docker-compose like below and use `docker-compose up` command:
```
version: "3"
services:
  hub:
    image: selenium/hub:3.14
    ports:
      - "4444:4444"
  chrome:
    image: selenium/node-chrome:3.14
    depends_on:
      - hub
    environment:
      - HUB_HOST=hub
  firefox:
    image: selenium/node-firefox:3.14
    depends_on:
      - hub
    environment:
      - HUB_HOST=hub
 ```
3.run `docker build -t your-repo:selenium-docker`. this will build a docker image of the automation framework. 
4.Test the docker image using run command by passing correct environemnt variables.

Using Jenkins for CICD: (brief description of tasks is mentioned below)

6.standup a Jenkins using docker command `docker run -p 8080:8080 -p 50000:50000 -v "$PWD/jenkins:/var/jenkins_home" jenkins/jenkins:lts`
7.stand up a Jenkins pipeline job that would read and execute below commands. (hosted in separate git repo)
```
pipeline {
    agent none
    stages {
        stage('Build Jar') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Image') {
            steps {
                script {
                	app = docker.build("your-repo/selenium-docker")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
			        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
			        	app.push("${BUILD_NUMBER}")
			            app.push("latest")
			        }
                }
            }
        }
    }
}
```
7. To run the tests via Jenkins, setup another pipeline job that would read and execute below commands. (hosted in separate git repo)
```
pipeline{
	agent any
	stages{
		stage("Pull Latest Image"){
			steps{
				sh "docker pull vinsdocker/selenium-docker"
			}
		}
		stage("Start Grid"){
			steps{
				sh "docker-compose up -d hub chrome firefox"
			}
		}
		stage("Run Test"){
			steps{
				sh "docker-compose up search-module book-flight-module"
			}
		}
	}
	post{
		always{
			archiveArtifacts artifacts: 'output/**'
			sh "docker-compose down"
			sh "sudo rm -rf output/"
		}
	}
}
```

 
