+++
Categories = ["agenda","workshop"]
Tags = ["logistics","speakers"]
date = "2016-03-15T16:23:23+09:00"
title = "Welcome to Pivotal Cloud Native Workshop @ HEB San Antonio"
type = "Introduction"
weight = 1
+++
##### Location

Corvette 2nd Floor Austin

##### Date and Time
Date: **June 22 , 2016**

Time: **9:00AM—1:00PM**

Click to get the Agenda, Prerequisites and Setup for the workshop.

<!--more-->

##### Agenda

        Broad Agenda (We will have breaks in between sessions)

        9:30 - 10:30: GM's Current and Desired Model of Software Delivery (3 Use Cases) -- Mano and GM Team

        10:30 - 11:30: Pivotal (and VMWare)'s approach to GM's desired state (Fulfilment of Use Cases)- Pivotal

        == LUNCH ==

        12:30 - 2:00: Walkthrough (and Hands on)  Push/Bind/Scale/Blue Green Deployment of Spring boot apps provided by Pivotal Cloud Foundry hosted by Pivotal

        2:00 - 3:00: Build Pipelines for continuous deployment, Bosh/Ops Manager walk through and discussion on CI/CD.

        3:30 - 4:30: Lessons learned and Scoping for POC - Apps, Infrastructure vSphere, Sizing, Timelines and Project Plan

        4:30 and beyond : Open discussion on various topics - Spring Cloud Services, Security, APM, K8, Docker
                          Wrap Up, Q&A, Feedback


##### Prerequisites
1. Java SDK 1.7+

2. Git CLI from [github.com](https://mac.github.com/)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Curl from [curl](http://curl.haxx.se/)

5. Use Workshop PCF Env or Pivotal Web Services Account.  Create a free trial account here [Pivotal Web Services](http://run.pivotal.io/)

6. Maven for build (https://maven.apache.org/install.html)

7. Vagrant (In case you want to run Concourse CI/CD using Vagrant)

##### Setup

1. Install the Prerequisites software (cf cli)

2. Check if you are able to use the cf cli to connect to the PCF Workshop Env. Alternatively, you can create a PWS account and check the firewall/connectivity before the Workshop

          cf login -a https://api.pcf2.cloud.fe.pivotal.io  --skip-ssl-validation

3. Check if you are able to connect to Git repo and download / clone the repo using CLI
4. Login to the App Manager Console at

        https://apps.pcf2.cloud.fe.pivotal.io

5. Note: Tiles preinstalled in the PCF Workshop environment which we will be using the workshop. You don't need any setup for these tiles.

          ER
          Spring Cloud Services
          RabbitMQ
          MySQL
          MongoDB
          Neo4J
          Redis


##### Speakers

    Marcelo Borges, Platform Architect
    Edward Mikuszewski, Platform Architect

#### Presentation


{{< googleslide "https://docs.google.com/presentation/d/1wNT7il4szv25Tl0KBG4SaetQkwKwyqk_rrQfskDSLgo/embed?start=false&loop=false&delayms=3000" >}}


#### Videos


{{< youtube V75fE_dxuBQ >}}


{{< youtube 7APZD0me1nU >}}
