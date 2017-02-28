+++

Categories = ["lab"]
Tags = ["cf","microservices","cloudfoundry"]
date = "2016-03-15T14:54:11-04:00"
title = "Lab : Build and Deploy Apps on PCF"
weight = 2

+++

### Goal


To deploy and configure a microservice and UI, leverage the platform for monitoring & management of the microservice, and do a blue green deployment with zero downtime.

<!--more-->

##### __PROXY_SETUP__

In case you run into Proxy issues, set the HTTP and HTTPS PROXY variable in the Global Environment, before you open a terminal window.

```
export HTTP_PROXY=<your http proxy>
export HTTPS_PROXY=<your https proxy>
```

For Windows
```
set HTTP_PROXY=<your http proxy>
set HTTPS_PROXY=<your https proxy>
```



Prerequisites
--

1. Java SDK 1.7+ [Java from Oracle](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

    Set the JAVA_HOME variable to the installation dir, in case it is not automatically set

2. Git CLI for [Windows](https://github.com/git-for-windows/git/releases/download/v2.9.0.windows.1/Git-2.9.0-64-bit.exe)
   or Git from [github.com](https://desktop.github.com)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Curl for [Windows](http://winampplugins.co.uk/curl/)
   Or for [Mac] (http://pdb.finkproject.org/pdb/package.php/curl)

5. Maven for build (https://maven.apache.org/install.html)

    Set the M2_HOME in case it is not automatically set
    Windows: set M2_HOME=<directory where Maven is installed>
    e.g.
    set M2_HOME=C:\Program Files\apache-maven-3.1.0-bin\apache-maven-3.1.0


7. Optional Golang (https://golang.org/dl/)  
   After you install Go, make sure you set the GOHOME and PATH
   For Linux/Mac:
    ```
    export GOROOT=$HOME/go
    export PATH=$PATH:$GOROOT/bin
    ```

    For Windows
    ```
    set GOROOT=C:\<Some directory>\go
    set PATH=%PATH%;%GOROOT%\bin
    ```


Steps
--

In this workshop we are going to follow these steps to deploy apps on Cloud foundry.

    - Get a Spring boot app from an existing Git Repository.
    - Deploy it to Pivotal Cloud foundry.
    - Manage the lifecycle of the application.

<img src="/images/blue-green.png" alt="DevOps on CF" style="width: 100%;"/>

__NOTE__

> The instructions in this document are for Mac/Linux based CLI/Shell. If you are using Windows, you may need to adjust your slashes.

***
## PART 1: Introduction to CF, Push an App.

### Step 1
##### Get the cities app

<img src="/images/git-clone.png" alt="Git style="width: 70%;"/>

Download the source code. Download as Zip file and save it in local folder

```bash
   unzip pcf-workspace-devops-master.zip
```
##### ---OR---

Fork and Clone

[PCF Workspace:  https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/](https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/)

For Linux/Mac:
```bash
$git clone https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops.git
```

For Windows
```
C:\<Some Directory to save code>\> git clone https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops.git
```







### Step 2
##### Build the app
By this point, you should have cloned (or forked, or downloaded) the [workspace repo](https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/).  Now you will build the project and deploy it to Cloud Foundry.

__NOTE__

In case you have HTTP_PROXY and HTTPS_PROXY create a gradle.properties file in the code directory

      systemProp.http.proxyHost=www.somehost.org
      systemProp.http.proxyPort=8080
      systemProp.http.proxyUser=userid
      systemProp.http.proxyPassword=password

For Linux/Mac:


    cd pcf-workspace-devops
    ./gradlew clean build


Windows:


    cd pcf-workspace-devops
    gradlew.bat clean build


### Step 3
##### Login into Pivotal Cloud Foundry

The students have userId's (student1-student25) and the passwords will be distributed in the workshop.
Each student is assigned an userId within their own organization (student1-org). Please refer to the handout you have been given.

````
cf login -a https://api.run.haas-98.pez.pivotal.io --skip-ssl-validation
  Email: student1
  Password: ••••••••

  Select a space (or press enter to skip):
  1. development
  2. test
  3. production

  Select any one and stick to that space for the rest of the workshop.

````

Login to the App Console at https://apps.run.haas-98.pez.pivotal.io

<img src="/images/pcf-console.png" alt="PCF App Console" style="width: 70%;"/>


### Step 4
##### Push the app


1. Push the cities-hello, put your initials in the app name so we don't get conflicts

    ```bash
    $ cd cities-hello
    $ cf push <studentXX>-cities-hello
    // This will give an output which is similar to this
    requested state: started
    instances: 1/1
    usage: 512M x 1 instances
    urls: cities-hello-lactiferous-unanswerableness.run.haas-98.pez.pivotal.io
    last uploaded: Mon Jun 15 14:53:10 UTC 2015
    stack: cflinuxfs2
    ```
2. Open the app url

    When you push the apps, it will give the url route to the app.
    <img src="/images/welcome.png" alt="Welcome to PCF Workshop" style="width: 70%;"/>

3. If you haven't already it is a good time to walk through the AppsManager:

        https://apps.run.haas-98.pez.pivotal.io

##### Recap: Part 1

> Cloud Foundry Haiku </br>
  Here is my source code </br>
  Run it on the cloud for me </br>
  I do not care how</br>

##### Discussion: Part 1

* How do you push an app to the cloud today?
* How does the cloud platform understand which runtime to use to run the app?

***
## PART 2: Push/Bind/Monitor/Scale


The cities-service app requires a database service to store and fetch cities info.

### Step 5
##### Create a Database from Marketplace

1. Review the docs on Marketplace Services:

    [Managing Services](http://docs.pivotal.io/pivotalcf/devguide/services/managing-services.html)

2. Create a mysql service, name it as `<YOUR INITIALS>-cities-db`

    You can create the service from the `cli` or launch the App Manager-> Select the Development Space [https://apps.run.haas-98.pez.pivotal.io](https://apps.run.haas-98.pez.pivotal.io) and login.
    Navigate to the marketplace and see the available services.

    <img src="/images/pcf-marketplace.png" alt="Marketplace Services" style="width: 70%;"/>
##### ---OR---
    You can create the service using the CLI.

    ````bash
    $ cf marketplace // check if mysql service is available
    $ cf create-service p-mysql 100mb-dev <studentXX>-cities-db
    ````

3. Launch the DB console via the `Manage` link in the App Manager.  Note the database is empty.

### Step 6
##### Push the App

1. Do a cf push on cities-service. Notice that the push will fail. In the next step you can learn why.

    ````bash
    $ cd ../cities-service  (on Windows cd ..\cities-service)
    $ cf push <studentXX>-cities-service -i 1 -m 512M -p build/libs/cities-service.jar
    ````
2. Check the logs to learn more about why the application is not starting.
    You can look at the recent logs from the cli or open up the App Console and view the log files for the app.

    ````bash
    $ cf logs <studentXX>-cities-service --recent
    ````
    <img src="/images/pcf-console-log.png" alt="Logs for the App" style="width:70%;"/>


### Step 7
##### Manually Binding the Service Instance

1. Review the docs on [Binding a Service Instance](http://docs.pivotal.io/pivotalcf/devguide/services/bind-service.html)
2. Bind the mysql instance `<YOUR INITIALS>-cities-db` to your app cities-service
    You can bind from the App Manager or from the `cli`

    ````bash
    $ cf bind-service <studentXX>-cities-service <studentXX>-cities-db
    ````

3. Restart your cities-service application to inject the new database.

    ````bash
    $ cf restart <studentXX>-cities-service
    ````

    Notice that the application is now running.

4. Check the Env variables to see if the service is bound.
    You can do it from App Manager or from the `cli`

    ````bash
    $ cf env <studentXX>-cities-service
    ````

5. Check the MySQL database through the `Manage` link in the App Manager to see that it now contains data.



__NOTE__

>    a. This app is an Spring Cloud app which uses Spring Cloud Configuration to bind to a database service provided by the cloud platform.

>    b. Difference between app `restage` and `restart`. An app `restage` will stop your application, run the application bits through the staging process to create a new droplet, and then start the new droplet.  `restart` will simply stop your application and start it with the existing droplet.  You typically `restart` when you need your application's' environment refreshed and you typically `restage` when you need/want the `buildpack` to run without updating the application source.

<br>

### Step 8
=======
##### Binding Services via the Manifest

Next, lets push the cities-service app with a manifest to help automate deployment.

1. Review the documentation: http://docs.pivotal.io/pivotalcf/devguide/deploy-apps/manifest.html
2. Edit the application manifest  `manifest.service` in your `cities-service`

    ````bash
    $ nano manifest.service
    ````

    On Windows

    ````bash
    notepad.exe manifest.service
    ````

3. Set the name of the app, the amount of memory, the number of instances, and the path to the .jar file.
*Be sure to name your application '<studentXX>-cities-service' e.g. student1-cities-service *
4. Add the services binding `<studentXX>-cities-db` to your deployment manifest for cities-service .
5. Test your manifest by re-pushing your app with no parameters:

    ````bash
    $ cf push -f manifest.service
    ````

    Notice that using a manifest, you have moved the command line parameters (number of instances, memory, etc) into the manifest.
6. Verify you can access your application via a curl request:
   You will have to get the route to your app

    ````bash
       // This will list your apps and the last column is the route.
       $ cf apps
          url: cities-hello-postpericardial-nonsubtlety.run.haas-98.pez.pivotal.io  
          // Note - Use HTTPS
       $ curl -i -k https://cities-hello-postpericardial-nonsubtlety.run.haas-98.pez.pivotal.io
    ````
    We must be able to access your application at https://cities-hello-postpericardial-nonsubtlety.run.haas-98.pez.pivotal.io for the next steps to work properly.

__NOTE__

> The default manifest file for an app is `manifest.yml` and it if is present, it is automatically picked without specifying the manifest file option.
In this exercise we have used a different naming convention.

<br>
### Step 7
##### PCF Metrics for Health, logging & events via the CLI

Learning about how your application is performing is critical to help you diagnose and troubleshoot potential issues. Cloud Foundry gives you options for viewing the logs.

Open the metrics dashboard at https://metrics.run.haas-98.pez.pivotal.io/
Use you login id/password which is your student id.

<img src="/images/pcf-metrics.png" alt="Metrics" style="width: 70%;"/>

You can Monitor your Container Metrics, Network Metrics and Events for your app. Explore your logs, which shows all your app logs streamed using the Loggregator.

<img src="/images/metrics-architecture.png" alt="Metrics" style="width: 70%;"/>

<br>
### Step 8
##### Environment variables

View the environment variable and explanation of [VCAP Env](http://docs.cloudfoundry.org/devguide/deploy-apps/environment-variable.html#view-env)

  ````bash
  $ cf env <studentXX>-cities-service
  ````


You will get the output similar to this on your terminal
  ````bash
  Getting env variables for app rj-cities-service in org Central / space development as rajesh.jain@pivotal.io...
  OK

  System-Provided:
  {
   "VCAP_SERVICES": {
    "cleardb": [
     {
      "credentials": {
       "hostname": "xxxx",
       "jdbcUrl": "xxxx",
       "name": "xxxx",
       "password": "xxxx",
       "port": "3306",
       "uri": "mysql://xxxx?reconnect=true",
       "username": "xxxx"
      },
      "label": "cleardb",
      "name": "rj-cities-db",
      "plan": "spark",
      "tags": [
       "Data Stores",
       "Cloud Databases",
       "Developer Tools",
       "Data Store",
       "mysql",
       "relational"
      ]
     }
    ]
   }
  }

  {
   "VCAP_APPLICATION": {
    "application_name": "rj-cities-service",
    "application_uris": [
     "rj-cities-service.haas-98.pez.pivotal.io"
    ],
    "application_version": "c3c35527-424f-4dbc-a4ea-115e1250cc5d",
    "limits": {
     "disk": 1024,
     "fds": 16384,
     "mem": 512
    },
    "name": "rj-cities-service",
    "space_id": "56e1d8ef-e87f-4b1c-930b-e7f46c00e483",
    "space_name": "development",
    "uris": [
     "rj-cities-service.haas-98.pez.pivotal.io"
    ],
    "users": null,
    "version": "c3c35527-424f-4dbc-a4ea-115e1250cc5d"
   }
  }

  User-Provided:
  SPRING_PROFILES_ACTIVE: cloud

  No running env variables have been set

  No staging env variables have been set
  ````


### Step 9
##### Scaling apps

Applications can be scaled via the command line or the console. When we talk about scale, there are two different types of scale: Vertical and Horizontal. Read [Scaling Apps](http://docs.cloudfoundry.org/devguide/deploy-apps/cf-scale.html) doc on more details on scaling applications.

When you vertically scale your application, you are increasing the amount of memory made available to your application. You would vertically scale your application while profiling your app, do performance tuning and to find the best memory settings before you deploy it in production.
Scaling your application horizontally means that you are adding application instances to increase your application throughput and performance under load.

Lets vertically scale the application to 1 GB of RAM.
  ````bash
  $ cf scale <studentXX>-cities-service -m 1G
  ````


Now scale your application down to 512 MB.

Next, lets scale up your application to 2 instances
  ````bash
  $ cf scale <studentXX>-cities-service -i 2
  ````


To check the status of your applications you can check from the command line to see how many instances your app is running and their current state
  ````bash
  $ cf app <studentXX>-cities-service
  ````


Once the second instance as started, scale the app back down to one instance.

You can also use the Autoscaler service from the marketplace and bind it to your app, so automatically scale it up or down.
<img src="/images/pcf-autoscaler.png" alt="Autoscaler" style="width: 70%;"/>


<br>
### Step 10
##### Verify the app from the Console

To verify that the application is running, use the following curl commands to retrieve data from the service or use a browser to access the URL:

  ````bash
  $ curl -i -k https://<studentXX>-cities-service.cfapps.haas-98.pez.pivotal.io/cities
  ````

  ````bash
  $ curl -i -k https://<studentXX>-cities-service.cfapps.haas-98.pez.pivotal.io/cities/49
  ````

  ````bash
  $ curl -i -k https://<studentXX>-cities-service.cfapps.haas-98.pez.pivotal.io/cities?size=5
  ````
<br>

##### Discussion: Part 2

In this part of the workshop we created a database service from the marketplace, pushed an app, bound it to the database service, monitored the health of the app and scaled the app.

1. How does the app get the database info today vs. VCAP_SERVICES? <br>
2. How do you horizontally scale your applications?



<br>

***
## PART 3: Deploying Upstream App and Bind to backend services

The `cities` directory also includes a `cities-ui` application which uses the `cities-client` to consume from the `cities-service`.

The `cities-client` demonstrates using the [Spring Cloud Connector](http://cloud.spring.io/spring-cloud-connectors) project to consume from a microservice.  This is a common pattern for Cloud Native apps.  For more details on building 12 Factor Apps for the Cloud (Cloud Foundry) refer to [12 Factor](http://12factor.net/) website.

The goal of this exercise is to use what you have learned to deploy the `cities-ui` application.

### Step 13
##### Build the Cities UI and Cities Client App


The cities-ui and cities-client can be both built at once by running `./gradlew assemble` in the parent directory. Run this command now if you did not run this in step # 2 above.


### Step 14
##### Create a User Provided Service Instance.

In this section we will create a backend microservice end point for cities-service.

1. Review the documentation on link:http://docs.pivotal.io/pivotalcf/devguide/services/user-provided.html[User Provided Service Instances]
2. Look for the details by running `cf cups --help`.

3. You will need to specify the parameter citiesuri to the user defined service instance .

  ````bash
  // Use the interactive prompt to create user defined service
  // It will prompt you for the parameters

  $ cf create-user-provided-service <studentXX>-cities-ws -p "citiesuri"

  citiesuri>   http://<studentXX>-cities-service.cfapps.haas-98.pez.pivotal.io/

  Creating user provided service....
  ````

<br>
### Step 15
##### Deploy cities-ui project


A `manifest.yml` is included in the cities-ui app.  Edit this manifest with your student number and add the service binding to your cities-service


  ````bash
  $ cd cities-ui
  $ nano manifest.yml (Or your favorite editor) (on Windows use notepad manifest.yml)

  ---
  applications:
  - name: <YOUR INITIALS>-cities-ui
    memory: 512M
    instances: 1
    path: build/libs/cities-ui.jar
    services: [ <YOUR INITIALS>-cities-ws ]
    env:
      SPRING_PROFILES_ACTIVE: cloud
  ````

Push the `cities-ui` without specifying the manifest.yml. It will by default pick the manifest.yml file and deploy the app.
  ````bash
  $ cf push
  ````

Note the URL once the application has been successfully pushed.

### Step 16
##### Verify the backend service is bound to cities-ui


````bash
----
$ cf env <studentXX>-cities-ui

System-Provided:
{
 "VCAP_SERVICES": {
  "user-provided": [
   {
    "credentials": {
     "tag": "cities",
     "uri": "https://rj-cities-service.haas-98.pez.pivotal.io/"
    },
    "label": "user-provided",
    "name": "cities-ws",
    "syslog_drain_url": "",
    "tags": []
   }
  ]
 }
}

{
 "VCAP_APPLICATION": {
  "application_name": "rj-cities-ui",
  "application_uris": [
   "rj-cities-ui.run.haas-98.pez.pivotal.io"
  ],
  "application_version": "dceb111b-3a68-45ad-83fd-3b8b836ebbe7",
  "limits": {
   "disk": 1024,
   "fds": 16384,
   "mem": 512
  },
  "name": "rj-cities-ui",
  "space_id": "56e1d8ef-e87f-4b1c-930b-e7f46c00e483",
  "space_name": "development",
  "uris": [
   "rj-cities-ui.run.haas-98.pez.pivotal.io"
  ],
  "users": null,
  "version": "dceb111b-3a68-45ad-83fd-3b8b836ebbe7"
 }
}

User-Provided:
SPRING_PROFILES_ACTIVE: cloud
````

### Step 17
##### Access the cities-ui to verify it is connected to your microservice.

Open the App Manager (Console) and navigate to your apps. You will see the cities-ui app, with a link to launch the cities-ui application. Alternatively you can open up your browser and navigate to the URL listed from a successful cf push command.


<img src="/images/cities-ui.png" alt="Cities UI" style="width: 100%;"/>




##### Discussion: Part 3


In this part of the workshop we created a cities-ui app which is loosely bound and independently developed from the backend service. We bound that app to the cities-service microservice.

1. Discussion on loose coupling of your services from your app and 12 Factor App design principles.


***
## PART 4: Deploy Version 2 of the App


In this section we are going to do a green-blue deployment using cf plugin `autopilot`. The same can be done by executing cf commands.

<br>

### Step 18
##### Process of Blue Green Deployment

  Review the CF Document for blue green deployment link:https://docs.pivotal.io/pivotalcf/devguide/deploy-apps/blue-green.html[Using Blue-Green Deployment to Reduce Downtime and Risk]

  In summary Blue-green deployment is a release technique that reduces downtime and risk by running two identical production environments called Blue and Green.


  <img src="/images/blue-green-process.png" alt="Blue Green Deployment Process" style="width: 100%;"/>


There are three different options in this lab to do blue-green deployment. You can pick any of them.

##### Option 1

First push a new version of the app with a blue route.

    // Delete the route studentXXX-cities-service if it is already mapped
    // $cf delete-route cfapps.haas-98.pez.pivotal.io --hostname studentXXX-cities-service

    $ cd cities-service
    // Push the app version v1 with the hostname as blue
    $ cf push studentXXX-cities-service-v1 --hostname studentXXX-cities-service-blue -f manifest.service
    // Map your outside route to this blue version
    $ cf map-route studentXXX-cities-service-v1 cfapps.haas-98.pez.pivotal.io --hostname studentXXX-cities-service
    $ cf apps // Check the apps and the routes

Next, you can push a new version of the app with a green route.

    // Push the app version v2 with the hostname as green
    $ cf push studentXXX-cities-service-v2 --hostname studentXXX-cities-service-green -f manifest.yml
    // Map the outside route to this green version. Now your outside route is mapped to both blue and green
    $ cf map-route studentXXX-cities-service-v2 cfapps.haas-98.pez.pivotal.io --hostname studentXXX-cities-service
    // Unmap the outside route to the blue version. All the traffic is now directed to v2
    $ cf unmap-route studentXXX-cities-service-v1 cfapps.haas-98.pez.pivotal.io --hostname studentXXX-cities-service


##### Option 2

Cloud Foundry plugin [Autopilot](https://github.com/concourse/autopilot) does blue green deployment, albeit it takes a different approach to other zero-downtime plugins. It does not perform any complex route re-mappings instead it leans on the manifest feature of the Cloud Foundry CLI. The method also has the advantage of treating a manifest as the source of truth and will converge the state of the system towards that. This makes the plugin ideal for continuous delivery environments.

Download the latest release of the autopilot plugin from the github releases page and save it in pcf-workspace-devops folder
 (https://github.com/contraband/autopilot/releases)

  On a Mac
  ````bash
  $ cd pcf-workspace-devops // This is the folder where you have saved the autopilot plugin
  $ cf install-plugin autopilot
  ````

  On Windows:

    cd pcf-workspace-devops // This is the folder where you have saved the autopilot plugin
    cf install-plugin autopilot.exe

  Once the plugin is installed ...

    $ cd cities-service (on Windows cd cities-service)
    // Append the build number to the app Name
    $ nano manifest.service // Change the app name and append the build number, on Windows use notepad manifest.service

<img src="/images/pcf-blue-green-b100.png" alt="Blue Green Deployment Build" style="width: 50%;"/>

    $ cf zero-downtime-push <studentXX>-cities-service -f manifest.service

##### Option 3 (This is as bash script and works only on Linux/OSX)

If you would like to inject build numbers in your app names here is a script you could use to do blue green deployments in the cities-service directory which only works on a Mac

    Usage: blue-green.sh <app-name> <build-number> <domain>


  ````bash

  $ ./blue-green.sh  cities-service 1001 run.haas-98.pez.pivotal.io
  $ cf apps // You should see your app build 1001 and the Route
  ````

  Now push the new build 1002 of the app

  ````bash
  $ ./blue-green.sh  cities-service 1002 run.haas-98.pez.pivotal.io
  $ cf apps // You should see your app build 1002 and the same route mapped to the new build

  ````

##### Discussion: Part 4

In this part of the workshop did deployment using a blue green script without any downtime.
This script / methodology can be used in your CD pipeline to build and deploy Cloud Native Apps with zero downtime.

    Discussion on how do you do Continuous Deployment and Delivery with zero downtime today.


##### Recap

In this workshop we saw how to build, deploy, bind, scale, monitor apps on Cloud foundry and manage the lifecycle of the application

<img src="/images/blue-green.png" alt="DevOps on CF" style="width: 100%;"/>


### Q/A

### Feedback


Please provide your feedback using this form [Feedback Form](https://docs.google.com/a/pivotal.io/forms/d/1qWlLtTuoULomw9DAW0tuhn7YVWXwVILaMTNKfXkcq0s/viewform?usp=send_form)
