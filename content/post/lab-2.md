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

***
## PART 1: Docker support in Pivotal Cloud Foundry

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

##### Discussion: Part 1

We have showed how docker images can run on Pivotal Cloud Foundry and compared and contrast with the buildpack approach of dynamically building the container images and managing in the platform. 

1. How are Docker images are created and patched? 

***
## PART 2: Running JBoss Docker image in Cloud Foundry

##### Prerequisites

1. Download workshop files [here](/files/lab2-part2-files.zip)

#### Step 1 - Review files

In the lab files folder you'll find a Dockerfile and apps folder with a war file. This is a stock Wildfly docker image which we're deploying the spring-music.war application.

#### Step 1 - Deploy Spring Music Application

We'll use the manifest file, manifest-bp.yml. You can review the file and issue the cf push command with the -f option as shown.

```bash
-> cat manifest-bp.yml 
---
applications:
- name: spring-music
  memory: 512M
  instances: 1
  host: spring-music
  path: apps/spring-music.war
  env:
    SPRING_PROFILES_DEFAULT: cloud
```

```bash
-> cf push -f manifest-bp.yml     
```

#### Step 2 - Create and bind a service

We'll use Pivotal Apps Manager to create and bind a MySQL service to the spring-music application. Click on the marketplace, then MySQL for Pivotal Cloud Foundry and select the 100mb-dev plan. Name your service instance, spring-music-db, add to your space and bind to your spring-music app.

<img src="/images/docker-bs.png" alt="DevOps on CF" style="width: 100%;"/>

You'll have to restart the application. You can check that the service is bound by clicking on the info icon on the upper, left corner at the application banner.

#### Step 3 - Deploy wildfly docker image

We can deploy a docker image using cf push with the -o option. Notice that we're defining a route path to the spring-music application context that we have deployed in the Dockerfile image. 

```bash
cf push spring-music-docker -o mborges/wildfly --route-path spring-music
```

The purpose is to lock down users from accessing the management console or other applications. We're using jboss wildly much like the tomcat embedded server in the Java buildpack.

All Pivotal Cloud Foundry capabilities are avaiable to this docker image. Log aggregation, self-healing, auto-scaling, HA and even the concept of services. 

Note: To access you'll have to use the complete path with /spring-music

#### Step 4 - Configure Management Port

The management port is protected, but we can access using SSH tunneling. That is the same method for doing remote debugging.

```bash
[-> cf ssh -N -T -L 9990:localhost:9990 spring-music-docker
```

Now you can go to your browser and access localhot:9990 and you'll be redirected to the jboss widly management console.


##### Discussion: Part 2

Pivotal Cloudry Foundry brings it's container management capabilities to docker with its has native. Application packaged as docker images can seemless integrate the ones using buildpacks. 

1. What are the benefits of wrapping an application as a docker image? 
2. What is the role of an application server in cloud native applications? 



