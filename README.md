# Docker-PageObjectModel_Selenium

This is a repositry that contains logic to get started with Selenium test auomtaion architecture following page object model design pattern. This readme file will help you provide idea on how to setup Jenkins to publish changes to architecture to the Docker hub when there are changes posted in gitlab and also to run tests in Jenkins. 

After cloning the project. Here are the steps to ensure that workspace is setup correctly.

1. run `mvn clean package -DSkipTests`. This will ensure that automation project and the dependencies are available as Jars under target folder.
2. Have selenium grid setup in a different location using a docker-compose like below:
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
3.run `docker build -t your-repo:selenium-docker`. this will build a docker image. 
4.Test the docker image using run command by passing correct environemnt variables.

 
