+++
Categories = ["lab"]
Tags = ["concourse","cloudfoundry"]
date = "2016-03-15T14:54:22-04:00"
title = "Lab 3: Build Pipelines using Concourse.ci"
weight = 5
+++


### Goal
In this workshop, you will learn how to Build Pipelines to for unit testing, staging and production deployment to Cloud Foundry using Concourse.ci

<!--more-->

### Introduction

Concourse's end goal is to provide an expressive CI system with as few distinct moving parts as possible.

Concourse CI decouples your project from your CI's details, and keeping all configuration in declarative files that can be checked into version control.

<img src="/images/concourse-1.png" alt="Concourse CI" style="width: 100%;"/>

Concourse limits itself to three core concepts: tasks, resources, and the jobs that compose them. Interesting features like timed triggers and synchronizing usage of external environments are modeled in terms of these, rather than as layers on top.

With these primitives you can model any pipeline, from simple (unit → integration → deploy → ship) to complex (testing on multiple infrastructures, fanning out and in, etc.).

Prerequisites
--

1. Java SDK 1.7+

2. Pivotal CF Env or Pivotal Web Services Account.  Create a free trial account here [Pivotal Web Services](http://run.pivotal.io/)

3. Vagrant (https://vagrantup.com/) to run Concourse locally

4. Fly cli. The fly tool is a command line interface to Concourse, it available when you bring up Concourse



Steps
--
In this workshop we are going to follow these steps to use the circuit-breaker in a Cloud Native Spring Boot app on Cloud foundry using the Spring Cloud Circuit Breaker Service.


Learn how to

    - Start and Configure Concourse.CI server
    - Create a Pipeline
    - Trigger a Pipeline using Fly
    - Run a pipeline to test, stage and deploy on Cloud Foundry


***

## Part 1: Building simple Pipelines

### Step 1
##### Configure your Concourse.CI server

Download the Concourse CI server and boot up using vagrant. This step will take some time, you can do this prior to the start of the workshop presentation.

````bash
$ mkdir ciworkshop && cd ciworkshop // ci workshop working directory
$ vagrant init concourse/lite # creates ./Vagrantfile
$ vagrant up                  # downloads the box and spins up the VM
````
The web server will be running at http://192.168.100.4:8080

Open up the Concourse UI web page, you don't have any pipelines configured. But you can download the fly cli from here. At the right hand bottom, use the links to download the **fly cli**.

If you're on Linux or OS X, you will have to `chmod +x` the downloaded binary and put it in your $PATH

Next, lets target and login to the Concourse server

````bash
$ fly -t lite login -c http://192.168.100.4:8080
````


### Step 2
##### Create your first pipeline

We have an existing project `flight-school` in a git repo, which we can clone and use for our first pipeline.

````
$ git clone https://github.com/mborges-pivotal/flight-school.git
$ cd flight-school\ci

````

In the ci folder there is a properties file, flight-school-properties-sample.yml

This contains the cf and git specific configuration which is read by the pipeline.

Make a local copy of the flight-school-properties-sample.yml


````
mkdir ~/.concourse
cp ci/flight-school-properties-sample.yml ~/.concourse/flight-school-properties.yml
chmod 600 ~/.concourse/flight-school-properties.yml
````

Now edit the ~/.concourse/flight-school-properties.yml

````
github-uri: https://github.com/.../flight-school.git
github-branch: master
cf-api: https://api.local.micropcf.io
cf-username: admin
cf-password: admin
cf-org: micropcf-org
cf-space: micropcf-space
cf-manifest-host: pcfdemo-ci

````

Review your pipeline file in the ci folder\pipeline.yml

````
resources:
- name: flight-school
  type: git
  source:
      uri: {{github-uri}}
      branch: {{github-branch}}
- name: staging-app
  type: cf
  source:
      api: {{cf-api}}
      username: {{cf-username}}
      password: {{cf-password}}
      organization: {{cf-org}}
      space: {{cf-space}}
      skip_cert_check: true

jobs:
- name: test-app
  plan:
  - get: flight-school
    trigger: true
  - task: tests
    file: flight-school/ci/tasks/build.yml
  - put: staging-app
    params:
      manifest: flight-school/manifest.yml

````

This pipeline has two resources and a single job. The resource `flight-school` is the `git` repo and the resource `staging-app` is the `cloud-foundry` space to stage the app.

The single job in the pipeline `test-app` gets the source code from git on any commits to the repo, and triggers the task defined in the `build.yml`. Next, on completion of this task, it puts the output artifact in to the staging-app resource using the manifest file defined for `cf push`

Edit the manifest file to reflect your app name

````
name: <student-id>-flight-school
memory: 128M
random-route: true
path: .

````


### Step 3
##### Set the Pipeline using Fly


Now you have your pipeline defined, it is ready to be uploaded to the CI Server.


````
$fly -t lite set-pipeline -p flight-school -c ci/pipeline.yml -l ~/.concourse/flight-school-properties.yml
````


### Step 4
##### Trigger the Pipeline and stage the app

Test the tasks manually before you run the whole Pipelines
````
fly -t lite execute -c ci/tasks/build.yml
````


````
fly -t lite trigger-job --job flight-school/test-app
````


<img src="/images/concourse-2.png" alt="Concourse CI" style="width: 100%;"/>


## Part 2: Running a real world pipeline

### Step 5
##### Configure a multi step pipeline

1. Clone the git repo which has a sample app PCFDemo with a real world pipeline.

    ````
    https://github.com/rjain-pivotal/PCF-demo
    ````

2. Make sure you have an S3 bucket configured to save your artifacts and the IAM user credentials to access the bucket.

3. Configure the properties files and assign it to the pipeline

    Copy the pcfdemo-properties-sample.yml to your ~/.concourse/pcfdemo-properties.yml
    Change the cf properties, github properties and s3 properties.

    ````
    github-uri: https://github.com/<github-user>/PCF-demo.git
    github-branch: master
    s3-access-key-id: SAMPLEDF99FSWEBF9DW9  # AWS or S3 compatible access key id
    s3-secret-access-key: sampleaxfdpiA98FG8u7ahd08Sdgf8AFG8gh8S0F  # AWS or S3 compatible secret access key
    s3-endpoint: s3.amazonaws.com
    s3-bucket-version: pcfdemo-releases
    s3-bucket-releases: pcfdemo-releases
    s3-bucket-release-candidates: pcfdemo-release-candidates
    maven-opts: # -Xms256m -Xmx512m
    maven-config: # -s path/to/settings.xml
    cf-api: https://api.local.micropcf.io
    cf-username: admin
    cf-password: admin
    cf-org: micropcf-org
    cf-space: micropcf-space
    cf-manifest-host: pcfdemo-ci
    ````



4. Set the pipeline


    ````
    fly -t lite set-pipeline -p pcfdemo -c ci/pipeline.yml -l ~/.concourse/pcfdemo-properties.yml
    ````

5. Trigger the pipeline


    ````
    fly -t lite trigger-job --job pcfdemo/unit-test
    ````



<img src="/images/concourse-3.png" alt="Concourse CI" style="width: 100%;"/>
