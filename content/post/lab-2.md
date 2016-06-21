+++
Categories = ["lab"]
Tags = ["docker","cloudfoundry"]
date = "2016-03-15T14:54:22-04:00"
title = "Lab 2: Run Docker Containers in Cloud Foundry"
weight = 2

+++

### Goal

To deploy and configure a spring boot app in Docker container and run it in Cloud Foundry.  
&nbsp;


<!--more-->

##### Prerequisites

1. Download workshop files [here](/files/lab2-files.zip)

2. Optional: Install [Docker Toolbox](https://www.docker.com/products/docker-toolbox) if you want to run the docker images locally - Step 3.

3. For this lab we'll use another PCF environment that has docker support enabled. Check which cloud you're target then login to a different cloud. 

   ```bash
     $ cf target
     $ cf login -a api.pcf.borgescloud.com --skip-ssl-validation -u <studentX> -p pivotal

     -> cf target
                
     API endpoint:   https://api.pcf.borgescloud.com (API version: 2.54.0)
     User:           student1
     Org:            HEB
     Space:          User1

   ```

You'll be placed into the HEB org and the UserX space.

#### Step 1 - Run it locally

   This is a very simple hello world Spring boot application packaged as a fatjar. 

   ```bash
     java -jar target/gs-spring-boot-docker-0.1.0.jar
   ```

#### Step 2 - Review Dockerfile

The Dockerfile simply runs the Spring Boot fatjar.

 ```bash
     FROM frolvlad/alpine-oraclejdk8:slim
     VOLUME /tmp
     ADD gs-spring-boot-docker-0.1.0.jar app.jar
     RUN sh -c 'touch /app.jar'
     ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
   ```

#### Step 3 - Containerize it and run locally - OPTIONAL

This is an optional step. If you have Docker Toolbox installed you can build the image, run it locally and optionally push to a docker hub. 

```bash
     docker build -t gs-spring-boot-docker .
     docker tag -f gs-spring-boot-docker mborges/gs-spring-boot-docker
     docker push mborges/gs-spring-boot-docker
   ```
 
   ```bash
      $ docker images
      # Get the Docker Machine VM IP if running on a MAC
      $ docker-machine env default (Get the Machine IP)
      $ docker run -p 8080:8080 -t mborges/gs-spring-boot-docker
      $ curl http://192.168.99.100:8080  (Use the Machine IP)
   ```

#### Step 4 - Run it in Cloud Foundry

Docker images can be deployed to Pivotal Cloud foundry using the cf push command.

   ```bash
      $cf push -o <docker-user>/gs-spring-boot-docker
   ```
