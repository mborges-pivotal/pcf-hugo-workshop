+++
Categories = ["lab"]
Tags = ["spring","circuit-breaker","cloudfoundry"]
date = "2016-04-15T11:28:20-04:00"
title = "Lab 5: Spring Cloud Circuit Breaker"
weight = 5
+++

### Goal
In this workshop, you will learn how to apply the circuit breaker pattern in your Spring boot apps.

&nbsp;

<!--more-->

### Introduction

Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer’s own laptop, bare metal data centers, and managed platforms such as Cloud Foundry.

The big picture : Use Spring Cloud Services design patterns to build cloud Native applications

<img src="/images/spring-1.png" alt="Cloud Native Spring Application Architecture" style="width: 100%;"/>


Circuit Breaker Dashboard for Pivotal Cloud Foundry® (PCF) provides Spring applications with an implementation of the Circuit Breaker pattern. Cloud-native architectures are typically composed of multiple layers of distributed services. End-user requests may comprise multiple calls to these services, and if a lower-level service fails, the failure can cascade up to the end user and spread to other dependent services. Heavy traffic to a failing service can also make it difficult to repair. Using Circuit Breaker Dashboard, you can prevent failures from cascading and provide fallback behavior until a failing service is restored to normal operation.

Prerequisites
--

1. Java SDK 1.7+

2. Git from [github.com](https://mac.github.com/)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Curl from [curl](http://curl.haxx.se/)

5. Pivotal Web Services Account.  Create a free trial account here [Pivotal Web Services](http://run.pivotal.io/)

6. Maven for build (https://maven.apache.org/install.html)


Steps
--
In this workshop we are going to follow these steps to use the circuit-breaker in a Cloud Native Spring Boot app on Cloud foundry using the Spring Cloud Circuit Breaker Service.


Learn how to


    - Config a Spring Cloud Circuit Breaker Service
    - Use Circuit Breaker  (Hysterix) in a Spring Boot application
    - Simulate a failure and watch the circuit breaker manage failures
    - Restore the service and watch the circuit breaker restore back the circuit


When applied to a service, a circuit breaker watches for failing calls to the service. If failures reach a certain threshold, it “opens” the circuit and automatically redirects calls to the specified fallback mechanism. This gives the failing service time to recover.

Desired the architecture of this Cloud Native Spring boot app is:

<img src="/images/circuit-breaker-1.png" alt="Circuit Breaker with Cloud Native Spring App" style="width: 100%;"/>



***

### Step 1
##### Get the traveler app

Clone the git repo which has a simple boilerplate Spring boot app built using Spring Initializer.

The Spring Labs repo contains multiple apps, we are going to focus on traveler app in this exercise.

````
git clone https://github.com/rjain-pivotal/pcf-workshop-spring-labs.git
````

### Step 2
##### Login into Pivotal Cloud Foundry

The students have userId's (student1-student25) and the passwords will be distributed in the workshop.
Each student is assigned their own Organization (student1-org)

````
cf login -a https://api.pcf2.cloud.fe.pivotal.io --skip-ssl-validation
  Email: student1
  Password: ••••••••
````

Login to the App Console at https://apps.pcf2.cloud.fe.pivotal.io

  <img src="/images/pcf-console.png" alt="PCF App Console" style="width: 100%;"/>



### Step 3
##### Configure the Spring Cloud Circuit Breaker Instance from the marketplace

1. In the PCF App Console, create a instance of the Registry Service from the marketplace.

      <img src="/images/pcf-console-2.png" alt="Marketplace Services" style="width: 100%;"/>

2. Select the default plan.
3. Name the service instance as 'studentXX-circuit-breaker-dashboard'

      <img src="/images/circuit-breaker-2.png" alt="Circuit Breaker" style="width: 100%;"/>

4. This will create the studentXX-circuit-breaker-dashboard service instance. To view the configuration of this service by clicking manage.

      <img src="/images/circuit-breaker-3.png" alt="Circuit Breaker" style="width: 100%;"/>


### Step 4

##### Code walk through (traveler)

Let's walk through the code in the traveler app in the source repo (Step #1) using your favorite editor (Atom/Sublime/Eclipse/IntelliJ/STS)

1. Review the *traveler/feign-agency/src/main/java/agency/AgencyApplication.java* file.

      To work with a Circuit Breaker Dashboard instance, your application must include the `@EnableCircuitBreaker` annotation on a configuration class. This client application also using service registry to discover the Company service and Feign to build the interace for accessing the services

      ````
      import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

      import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;

      import org.springframework.cloud.netflix.feign.EnableFeignClients;

      @SpringBootApplication
      @EnableDiscoveryClient
      @RestController
      @EnableCircuitBreaker
      @EnableFeignClients
      public class AgencyApplication {
      ````

2. Review the *traveler/feign-agency/pom.xml* file. By adding *spring-cloud-services-starter-circuit-breaker* to the classpath this application is able to use the circuit breaker.

      ````
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
      </dependency>

      <dependency>
        <groupId>io.pivotal.spring.cloud</groupId>
        <artifactId>spring-cloud-services-starter-service-registry</artifactId>
      </dependency>

      <dependency>
        <groupId>io.pivotal.spring.cloud</groupId>
        <artifactId>spring-cloud-services-starter-circuit-breaker</artifactId>
      </dependency>

      ````

3. Review the *traveler/feign-agency/src/main/java/agency/TravelAgent.java*

      To apply a circuit breaker to a method, annotate the method with ```@HystrixCommand```, giving the annotation the name of a fallbackMethod.

        @HystrixCommand(fallbackMethod = "getBackupGuide")
        public String getGuide() {
            return company.availableGuide();
        }

        String getBackupGuide() {
            return "None available! Your backup guide is: Cookie";
        }

      The `getGuide()` method uses a RestTemplate to obtain a guide name from another application called Company, which is registered with a Service Registry instance.

      The method thus relies on the Company application to return a response, and if the Company application fails to do so, calls to `getGuide()` will fail. When the failures exceed the threshold, the breaker on `getGuide()` will open the circuit.

      While the circuit is open, the breaker redirects calls to the annotated method, and they instead call the designated fallbackMethod. The fallback method must be in the same class and have the same method signature (i.e., have the same return type and accept the same parameters) as the annotated method.

For more details, refer to the documentation of the Circuit Breaker configuration here (https://docs.pivotal.io/spring-cloud-services/circuit-breaker/writing-client-applications.html)

### Step 5
##### Push the app to cloud Foundry

1. Change the manifest.yml file in the traveler/company to reflect the name of the app and the service-registry

        ---
        memory: 512M
        applications:
          - name: <studentXX>-company
            services:
              - <studentXX>-service-registry
            path: ./target/company-0.0.1-SNAPSHOT.jar
            env:
              SPRING_PROFILES_ACTIVE: dev
              CF_TARGET: https://api.pcf2.cloud.fe.pivotal.io


2. Change the manifest.yml file in the traveler/feign-agency to reflect the name of the app and the circuit-breaker and service-registry Services


        ---
        instances: 1
        memory: 512M
        applications:
          - name:  <studentXX>-agency
            path: ./target/agency-0.0.1-SNAPSHOT.jar
            services:
              -  <studentXX>-service-registry
              -  <studentXX>-circuit-breaker-dashboard
            env:
              SPRING_PROFILES_ACTIVE: dev
              CF_TARGET: https://api.pcf2.cloud.fe.pivotal.io



3. Build the app using maven in the parent traveler directory

      ````
      $cd traveler
      $mvn clean package
      ````

4. Push the apps using scripts/deploy_mvn.sh or scripts/deploy_mvn.bat

    First check and change the service names in the script. If the script registry is already created don't create a new one.


      ````
      #cf create-service p-service-registry standard <studentXX>-service-registry
      cf create-service p-circuit-breaker-dashboard standard <studentXX>-circuit-breaker-dashboard
      sleep 120
      pushd company && cf push -p target/company-0.0.1-SNAPSHOT.jar
      popd; sleep 30
      pushd feign-agency && cf push -p target/agency-0.0.1-SNAPSHOT.jar
      popd
      echo "" && echo "Done!" && echo ""
      ````

    Now, push using this scripts

      ````bash
      $cd traveler
      $./scripts/deploy_mvn.sh
      ````

5. Open in the browser the App

    First, Get the route to your app

      ````
      // This is the agency app
      http://<studentXX>-agency.pcf2.cloud.fe.pivotal.io/

      // Note this is the company app
      http://<studentXX>-company.pcf2.cloud.fe.pivotal.io/available
      ````

6.  Check the Hysterix Dashboard from the App Console -> Manage Hysterix Service instance

      When service calls are succeeding, the circuit is closed, and the dashboard graph shows the rate of calls per second and successful calls per 10 seconds.   

      <img src="/images/circuit-breaker-4.png" alt="Circuit Breaker Open" style="width: 100%;"/>


### Step 6
##### Shut down the Company Service

1. If the Company application becomes unavailable or if the Agency application cannot access it, calls to getGuide() will fail.

      When successive failures build up to the threshold, Hystrix will open the circuit, and subsequent calls will be redirected to the getBackupGuide() method until the Company application is accessible again and the circuit is closed.

        $cf stop  <studentXX>-company

      Now check the app status, the agency app will fall back to the backup.

        http:// <studentXX>-agency.pcf2.cloud.fe.pivotal.io/

        Your guide will be: None available! Your backup guide is: Cookie

2. Check the Hysterix Dashboard from the App Console -> Manage Hysterix Service instance

      When calls begin to fail, the graph shows the rate of failed calls in red.


      <img src="/images/circuit-breaker-5.png" alt="Circuit Breaker Open" style="width: 100%;"/>

      Load the circuit

        while true; do curl http://<studentXX>-agency.pcf2.cloud.fe.pivotal.io/; done


3. When failures exceed the configured threshold (the default is 20 failures in 5 seconds), the breaker opens the circuit.

      The dashboard shows the rate of short-circuited calls—calls which are going straight to the fallback method—in blue.

      The application is still allowing calls to the failing method at a rate of 1 every 5 seconds, as indicated in red; this is necessary to determine if calls are succeeding again and if the circuit can be closed.

      <img src="/images/circuit-breaker-6.png" alt="Circuit Breaker Open" style="width: 100%;"/>
