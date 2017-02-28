+++
Categories = ["agenda","workshop"]
Tags = ["logistics","speakers"]
date = "2016-03-15T16:23:23+09:00"
title = "Welcome to the Pivotal Cloud Foundry Workshop@TOLA"
type = "Introduction"
weight = 1
+++

##### Date and Time
Date: **Nov 15 , 2016**

Time: **8:00AM - 4:00PM**

Click to get the Agenda, Prerequisites and Setup for the workshop.

<!--more-->

#### Agenda
|  |  |
|------|------|
| **8:00 AM–9:00 AM** | Registration and Breakfast  |
| **9:00 AM–10:00 AM** | Cloud Native Architecture & Pivotal Cloud Foundry |    
| **10:15 AM–12:00 PM** | Hands-On Experience with Pivotal Cloud Foundry  (push, bind, scale, monitor) |
| **12:00 PM–1:30 PM** | Lunch and Networking  |
| **1:30 PM–3:30 PM** | CI/CD on Cloud Foundry using Concourse  |
| **3:30 PM–4:00 PM** | Wrap Up, Q&A, Feedback  |

---

##### Prerequisites
1. Java SDK 1.7+ [Java from Oracle](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

          Set the JAVA_HOME variable to the installation dir, in case it is not automatically set

2. Git CLI for [Windows](https://github.com/git-for-windows/git/releases/download/v2.9.0.windows.1/Git-2.9.0-64-bit.exe)
   or Git from [github.com](https://desktop.github.com)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Curl for [Windows](http://winampplugins.co.uk/curl/)
   Or for [Mac] (http://pdb.finkproject.org/pdb/package.php/curl)

##### Setup

1. Install the Prerequisites software (cf cli)

2. Check if you are able to use the cf cli to connect to the PCF Workshop Env. Alternatively, you can create a PWS account and check the firewall/connectivity before the Workshop

          cf login -a https://api.run.haas-68.pez.pivotal.io  --skip-ssl-validation

3. Check if you are able to connect to Git repo and download / clone the repo using CLI
4. Login to the App Manager Console at

        https://apps.pcf2.cloud.fe.pivotal.io

5. Note: Tiles preinstalled in the PCF Workshop environment which we will be using the workshop. You don't need any setup for these tiles.

          ER
          RabbitMQ
          MySQL
          Redis

        https://console.run.pivotal.io


#### Speakers
+ Marcelo Borges - Platform Architect at Pivotal

#### Presentation

{{< googleslide "https://docs.google.com/presentation/d/1YZnnp580XMTRCyKYWxKjaR_G53yvD46jBwhfsE0mKbQ/embed?start=false&loop=false&delayms=3000" >}}


#### Videos


{{< youtube xdw_9dADM-4 >}}


#### EBooks
Beyond the Twelve-Factor App - https://pivotal.io/beyond-the-twelve-factor-app
Cloud Foundry: The Cloud Native Platform - https://pivotal.io/cloud-foundry-the-cloud-native-platform
Migrating to Cloud Native Application Architectures - https://pivotal.io/platform/migrating-to-cloud-native-application-architectures-ebook
