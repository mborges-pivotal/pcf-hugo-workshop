+++
Categories = ["lab"]
Tags = ["apigee","cloudfoundry"]
date = "2016-06-24T14:54:22-04:00"
title = "Lab : Routing Services and API Management using Apigee Service"
weight = 4
+++


### Goal
In this workshop, you will learn how to use Apigee Edge Service to do unit testing, staging and production deployment to Cloud Foundry using Concourse.ci

<!--more-->

### Introduction

With Apigee Edge's integration with Pivotal Cloud Foundry (PCF), you can have access to full API management functionality between clients and your Cloud Foundry application. With a single command from the cf CLI, you can create a new Edge API proxy that is immediately ready to handle requests sent to an existing Cloud Foundry application. The presence of the proxy is seamless to your Cloud Foundry application.

<img src="/images/apigee-1.png" alt="Apigee Edge Service" style="width: 100%;"/>


Once the API proxy is created, you can enhance it with more functionality and services, including:

1. Data transformation, mediation, orchestration, policies for rate limits, and so on.
2. Security, such as authentication and RBAC.
3. Onboarding, including a developer portal to enable self-service.
4. Analytics for monitoring and business metrics analysis.
5. Monetization for rate plans, internationalization, and usage tracking.




Prerequisites
--

1. Java SDK 1.7+

2. Pivotal CF Env with Apigee Service Tile

3. Apigee Account at (https://accounts2.apigee.com/accounts/sign_up)

4. Activate API Management from your dashboard after clicking on activation email




Steps
--
In this workshop we are going to follow these steps to use the Apigee Edge Service to proxy route requests to an app deployed on PCF and perform API gateway steps on the request.

Learn how to

    - Configure Apigee Service Tile and Service
    - Create an app and bind it to the Apigee Service
    - Perform Route requests and watch Apigee Service proxy the requests
    - Perform Rate Limiting on the Route


***

## Part 1: Create an app and bind to Apigee Service
### Step 1
##### Clone Sample App

Clone the Apigee Pivotal Cloud Foundry sample at the following URL:

https://github.com/apigee/pivotal-cf-apigee/tree/master/sample-api

For example, from the command-line you can clone the repository this way:


    $ git clone https://github.com/apigee/pivotal-cf-apigee.git
    $ cd pivotal-cf-apigee/sample-api

### Step 2
##### Push it to Cloud Foundry

Edit manifest.yml to change the name and host properties to something specific for you.
*Be sure to name your application `<studentXX>-apigee-demo` *

    ---
    applications:
    - name: <studentXX>-apigee-demo
      memory: 128M
      instances: 1
      host: <studentXX>-apigee-demo
      path: .
      buildpack: nodejs_buildpack

Push the app to cloud Foundry

    $ cf push
    $ cf apps // This will list your app. Get the route to your app


    Getting apps in org student25-org / space development as instructor...
    OK

    name                    requested state   instances   memory   disk   urls
    student28-apigee-demo   started           1/1         128M     1G     student28-apigee-demo.pcf2.cloud.fe.pivotal.io

    //The route to your app is https://student28-apigee-demo.pcf2.cloud.fe.pivotal.io

Send a request to the app route using `curl`

    $ curl -k https://<studentXX>-apigee-demo.pcf2.cloud.fe.pivotal.io

    // Output
    {"hello":"hello from cf app"}


### Step 3
##### Create Apigee Service and bind to the app

We have already configured the Apigee Service Tile from the Pivotal Network

https://network.pivotal.io/products/apigee-edge-for-pcf-service-broker

Login to the the Apigee Enterprise console

https://enterprise.apigee.com/platform/

Get the Org Name, Env, User Id, Password

<img src="/images/apigee-2.png" alt="Apigee Enterprise Console" style="width: 100%;"/>


Create the Apigee service instance using the Apigee Marketplace service

    $ cf marketplace

    Getting services from marketplace in org student25-org / space development as instructor...
    OK

    service                       plans                     description
    apigee-edge                   org                       Apigee Edge API Platform

    // Create a unique name to your service, replace <studentXXX>-api-connectors-service with your studentID.
    $ cf create-service apigee-edge org <studentXXX>-api-connectors-service -c '{"org":"<your org>", "env":"<your env>", "user":"<your user id>", "pass":"<your password>", "host": "apigee.net", "hostpattern": "${apigeeOrganization}-${apigeeEnvironment}.${proxyHost}"}'

    // windows syntax
    cf create-service apigee-edge org <studentXXX>-api-connectors-service -c "{\"org\":\"<your org>\", \"env\":\"<your env>\", \"user\":\"<your user id>\", \"pass\":\"<your password>\", \"host\": \"apigee.net\", \"hostpattern\": \"${apigeeOrganization}-${apigeeEnvironment}.${proxyHost}\"}"

    // NOTE: Only replace the values above that are enclosed in <brackets>. The first org listed actually refers to the Apigee-edge org plan.

    //The JSON specifies the Apigee Edge details needed to route traffic:

    org -- Change to your org, Organization of the Apigee Edge proxy through which requests should be routed.
    env -- Change to your Env, Environment of the Apigee Edge proxy through which requests should be routed.
    user -- Change to your User, Username of an Edge user who has access to create proxies.
    pass -- Change to your Password, Password of an Edge user who has access to create proxies.
    host -- Don't change. Edge host name to which requests to your API proxies can be sent.
    hostpattern -- Don't change. Pattern for generating the API proxy URL. For example,
                  #{apigeeOrganization}-#{apigeeEnvironment}.#{proxyHost} for cloud accounts.


Use the bind-route-service command to create an Edge API proxy and bind your Cloud Foundry application to the proxy.

This tells the Go router to redirect requests to the Apigee Edge proxy before sending them to the Cloud Foundry application.

    $ cf bind-route-service pcf2.cloud.fe.pivotal.io <studentXXX>-api-connectors-service --hostname <studentXX>-apigee-demo

### Step 4
##### Send Requests and Watch Apigee Edge Service


Now you have your app route bound to the Apigee Service, lets send some requests and watch the Apigee Edge Service proxy the requests.

First, In the Apigee management console, under APIs > API proxies, locate the name of the proxy you just created.

<img src="/images/apigee-3.png" alt="Apigee Edge Service" style="width: 100%;"/>



Click the PCF proxy's name to view its overview page.

<img src="/images/apigee-4.png" alt="Apigee Edge Service" style="width: 100%;"/>


Click the Trace tab, the click the Start Trace Session button.

<img src="/images/apigee-5.png" alt="Apigee Edge Service" style="width: 100%;"/>


Back at the command line, run the cURL command you ran earlier -- the one that makes a request to the Cloud Foundry application you pushed.

    $ curl -k https://<studentXXX>-apigee-demo.pcf2.cloud.fe.pivotal.io

As before, the console will display the app's response.

    {"hello":"hello from cf app"}


Return to the Edge management console to see that your request has now been routed through the Edge proxy you created.

<img src="/images/apigee-6.png" alt="Apigee Edge Service" style="width: 100%;"/>


Finally, Stop the Tracing Session


## Part 2: Add Apigee API Policies

### Step 5
##### Configure API Policies

Go the Apigee Management Console and Switch to the Development Menu.

<img src="/images/apigee-7.png" alt="Apigee Edge Service" style="width: 100%;"/>



#####  Add Policy
Click on Add Step, this will pop-up a list of Apigee Policies

<img src="/images/apigee-8.png" alt="Apigee Edge Service" style="width: 100%;"/>


##### Select Spike Arrest

This policy will arrest if there is a spike in load from the same session. Change the Rate from default 30 fps to 3 fpm (3 Requests per minute) to trigger the policy.


<img src="/images/apigee-9.png" alt="Apigee Edge Service" style="width: 100%;"/>

#####  Click Save

Now, run multiple cURL commands from the command line. Run the cURL command you ran earlier -- the one that makes a request to the Cloud Foundry application you pushed, more than three times in a minute

    $ curl -k https://<studentXXX>-apigee-demo.pcf2.cloud.fe.pivotal.io

##### Spike Error

Notice the error coming back from the curl command.

    $curl -k https://student28-apigee-demo.pcf2.cloud.fe.pivotal.io
      {"fault":{"faultstring":"Spike arrest violation. Allowed rate : 3pm","detail":{"errorcode":"policies.ratelimit.SpikeArrestViolation"}}}
