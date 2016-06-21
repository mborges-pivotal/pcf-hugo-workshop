+++

Categories = ["lab"]
Tags = ["cf","microservices","cloudfoundry"]
date = "2016-03-15T14:54:11-04:00"
title = "Lab 1: Build and Deploy Apps on PCF"
weight = 1

+++

### Goal


To deploy and configure a microservice and UI, leverage the platform for monitoring & management of the microservice, and do a blue green deployment with zero downtime.

<!--more-->


Prerequisites
--

1. Access to Pivotal Cloud Foundry Environment

2. Download workshop files [here](/files/lab1-files.zip)



Steps
--
In this workshop we are going to follow these steps to deploy apps on Cloud foundry and manage the lifecycle of the application.

<img src="/images/devops-cf.png" alt="DevOps on CF" style="width: 100%;"/>

Learn how to

    - Get a Spring boot app and deploy it to Pivotal Cloud foundry

__NOTE__

> The instructions in this document are for Mac/Linux based CLI/Shell. If you are using Windows, you may need to adjust your slashes.

***
## PART 1: Introduction to CF, Push an App.

### Step 1
##### Get the cities app
1. Extract the application binaries and cd into its folder

### Step 2
##### Login into Pivotal Cloud Foundry

The students have userId's (student1-student25) and the passwords will be distributed in the workshop.
Each student is assigned an userId within their own organization (student1-org). Please refer to the handout you have been given.

````
cf login -a https://api.run.pivotal.io
  Email: student1
  Password: ••••••••
````

Login to the App Console at https://apps.run.pivotal.io

<img src="/images/pcf-console.png" alt="PCF App Console" style="width: 100%;"/>

### Step 3
##### Push the app


1. Push the cities-hello, put your initials in the app name so we don't get conflicts

    ```bash
    $ cd cities-hello
    $ cf push <studentXX>-cities-hello
    // This will give an output which is similar to this
    requested state: started
    instances: 1/1
    usage: 512M x 1 instances
    urls: cities-hello-lactiferous-unanswerableness.pcf2.cloud.fe.pivotal.io
    last uploaded: Mon Jun 15 14:53:10 UTC 2015
    stack: cflinuxfs2
    ```
2. Open the app url

    When you push the apps, it will give the url route to the app.
    <img src="/images/welcome.png" alt="Welcome to PCF Workshop" style="width: 100%;"/>

3. If you haven't already it is a good time to walk through the AppsManager:

        https://apps.run.pivotal.io

##### Recap: Part 1

> Cloud Foundry Haiku </br>
  Here is my source code </br>
  Run it on the cloud for me </br>
  I do not care how</br>


##### Discussion: Part 1
+ How do you push an app to the cloud today?
+ How does the cloud platform understand which runtime to use to run the app?

***
## PART 2: Push/Bind/Monitor/Scale


The cities-service app requires a database service to store and fetch cities info.

### Step 4
##### Create a Database from Marketplace

1. Review the docs on Services:

    [Managing Services](http://docs.pivotal.io/pivotalcf/devguide/services/managing-services.html)

2. Create a mysql service, name it as `<YOUR INITIALS>-cities-db`

    You can create the service from the `cli` or launch the App Manager-> Select the Development Space [https://apps.run.pivotal.io](https://apps.run.pivotal.io) and login.

    Navigate to the marketplace and see the available services. Here you will create the service using the CLI.
  ````bash
    $ cf marketplace // check if mysql service is available
    $ cf cf create-service cleardb spark <studentXX>-cities-db
  ````

3. Launch the DB console via the `Manage` link in the App Manager.  Note the database is empty.

### Step 5
##### Push the App

1. Do a cf push on cities-service. **NOTICE THAT THE PUSH WILL FAIL.** In the next step you can learn why.

    ````bash
    $ cd ../cities-service
    $ cf push <studentXX>-cities-service -i 1 -m 512M -p build/libs/cities-service.jar
    ````
2. Check the logs to learn more about why the application is not starting
    You can look at the recent logs from the cli or open up the App Console and view the log files for the app.

    ````bash
    $ cf logs <studentXX>-cities-service --recent
    ````
    <img src="/images/pcf-console-log.png" alt="Logs for the App" style="width: 100%;"/>


### Step 6
##### Manually Binding the Service Instance

1. Review the docs on [Binding a Service Instance](http://docs.pivotal.io/pivotalcf/devguide/services/bind-service.html)
2. Bind the mysql instance `<YOUR INITIALS>-cities-db` to your app cities-service
    You can bind from the App Manager or from the `cli`

    ````bash
    $ cf bind-service <studentXX>-cities-service <studentXX>-cities-db
    ````

3. Restart your cities-service application to inject the new database. Notice the CF CLI may suggest you restage the application, but this is not required in this case.

    ````bash
    $ cf restart <studentXX>-cities-service
    ````

    Notice that the application is now running.

4. Check the Env variables to see if the service is bound.
    You can do it from App Manager or from the `cli`

    ````bash
    $ cf env <studentXX>-cities-service
    ````

5. Check the MySQL database to see that it now contains data using MySQL Workbench or a similar tool.



__NOTE__

>    a. This app is an Spring Cloud app which uses Spring Cloud Configuration to bind to a database service provided by the cloud platform.

>    b. Difference between app `restage` and `restart`. An app `restage` will stop your application, run the application bits through the staging process to create a new droplet, and then start the new droplet.  `restart` will simply stop your application and start it with the existing droplet.  You typically `restart` when you need your application's' environment refreshed and you typically `restage` when you need/want the `buildpack` to run without updating the application source.

<br>

### Step 7
##### Binding Services via the Manifest

Next, lets push the cities-service app with a manifest to help automate deployment.

1. Review the documentation: http://docs.pivotal.io/pivotalcf/devguide/deploy-apps/manifest.html
2. Edit the application manifest  `manifest.service` in your `cities-service`

    ````bash
    $ nano manifest.service
    ````

3. Set the name of the app, the amount of memory, the number of instances, and the path to the .jar file.
*Be sure to name your application '<studentXX>-cities-service' *
4. Add the services binding `<YOUR INITIALS>-cities-db` to your deployment manifest for cities-service .
5. Now, manually unbind the service and re-push your app using the manifest.

    ````bash
    $ cf unbind-service <studentXX>-cities-service <studentXX>-cities-db
    ````


6. Test your manifest by re-pushing your app with no parameters:

    ````bash
    $ cf push -f manifest.service
    ````

    Notice that using a manifest, you have moved the command line parameters (number of instances, memory, etc) into the manifest.
7. Verify you can access your application via a curl request:
   You will have to get the route to your app

    ````bash
       // This will list your apps and the last column is the route.
       $cf apps
          url: student1-cities-service.cfapps.io  
          $ curl -i http://student1-cities-service.cfapps.io
    ````
    We must be able to access your application at  http://student1-cities-service.cfapps.io for the next steps to work properly.

__NOTE__

> The default manifest file for an app is `manifest.yml` and it if is present, it is automatically picked without specifying the manifest file option.
In this exercise we have used a different naming convention.

<br>
### Step 8
##### Health, logging & events via the CLI

Learning about how your application is performing is critical to help you diagnose and troubleshoot potential issues. Cloud Foundry gives you options for viewing the logs.

To tail the logs of your application perform this command:
  ````bash
  $ cf logs <studentXX>-cities-service
  ````


Notice that nothing is showing because there isn't' any activity. Use the following curl command to see the application working:
  ````bash
  $ curl -i http://<studentXX>-cities-service.cfapps.io/cities/
  ````

For other ways of viewing logs check out the documentation here: [Streaming Logs](http://docs.pivotal.io/pivotalcf/devguide/deploy-apps/streaming-logs.html#view)

To view recent events, including application crashes, and error codes, you can see them from the App Manager or from the cli.

  ````bash
  $ cf events <studentXX>-cities-service
  ````

To view the health of the application you can see from the App Manager or from the cli:
  ````bash
  $ cf app <studentXX>-cities-service
  ````

You will get detailed output of the health
  ````bash
  Showing health and status for app cities-service in org  / space development as...
  OK

  requested state: started
  instances: 1/1
  usage: 512M x 1 instances
  urls: cities-service.pcf2.cloud.fe.pivotal.io
  last uploaded: Wed May 27 15:53:32 UTC 2015
  stack: cflinuxfs2

       state     since                    cpu    memory           disk           details
  #0   running   2015-05-27 12:17:55 PM   0.1%   434.5M of 512M   145.4M of 1G
  ````

<br>
### Step 9
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
     "rj-cities-service.pcf2.cloud.fe.pivotal.io"
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
     "rj-cities-service.pcf2.cloud.fe.pivotal.io"
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


### Step 10
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

<br>
### Step 11
##### Verify the app from the Console

To verify that the application is running, use the following curl commands to retrieve data from the service or use a browser to access the URL:

  ````bash
  $ curl -i http://<studentXX>-cities-service.cfapps.io/cities
  ````

  ````bash
  $ curl -i http://<studentXX>-cities-service.cfapps.io/cities/162
  ````

  ````bash
  $ curl -i http://<studentXX>-cities-service.cfapps.io/cities?size=5
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

### Step 12
##### Create a User Provided Service Instance.

In this section we will create a backend microservice end point for cities-service.

1. Review the documentation on link: [User Provided Service Instances](http://docs.pivotal.io/pivotalcf/devguide/services/user-provided.html)
2. Look for the details by running `cf cups --help`.

3. You will need to specify the parameter citiesuri to the user defined service instance .

  ````bash
  // Use the interactive prompt to create user defined service
  // It will prompt you for the parameters

  $ cf create-user-provided-service <studentXX>-cities-ws -p "citiesuri"

  citiesuri>   http://<studentXX>-cities-service.cfapps.io/

  Creating user provided service....
  ````

<br>
### Step 13
##### Deploy cities-ui project


A `manifest.yml` is included in the cities-ui app.  Edit this manifest with your initials and add the service binding to your cities-service


  ````bash
  $ cd cities-ui
  $ nano manifest.yml (Or your favorite editor)

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

### Step 14
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
     "uri": "http://rj-cities-service.pcf2.cloud.fe.pivotal.io/"
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
   "rj-cities-ui.pcf2.cloud.fe.pivotal.io"
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
   "rj-cities-ui.pcf2.cloud.fe.pivotal.io"
  ],
  "users": null,
  "version": "dceb111b-3a68-45ad-83fd-3b8b836ebbe7"
 }
}

User-Provided:
SPRING_PROFILES_ACTIVE: cloud
````

### Step 15
##### Access the cities-ui to verify it is connected to your microservice.

Open the App Manager (Console) and navigate to your apps. You will see the cities-ui app, with a link to launch the cities-ui application. Alternatively you can open up your browser and navigate to the URL listed from a successful cf push command.


<img src="/images/cities-ui.png" alt="Cities UI" style="width: 100%;"/>




##### Discussion: Part 3


In this part of the workshop we created a cities-ui app which is loosely bound and independently developed from the backend service. We bound that app to the cities-service microservice.

1. Discussion on loose coupling of your services from your app and 12 Factor App design principles.


***
## PART 4: Deploy Version 2 of the App


##### THIS PART IS BEING UPDATED
<br>

***
### Recap

In this workshop we saw how to build, deploy, bind, scale, monitor apps on Cloud foundry and manage the lifecycle of the application

<img src="/images/devops-cf.png" alt="DevOps on CF" style="width: 100%;"/>


### Q/A

### Feedback


Please provide your feedback using this form [Feedback Form](https://docs.google.com/a/pivotal.io/forms/d/1qWlLtTuoULomw9DAW0tuhn7YVWXwVILaMTNKfXkcq0s/viewform?usp=send_form)
