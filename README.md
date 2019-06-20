# cdc-pact-expert-talk
This talk was presented by [Prashant Kalkar](https://github.com/prashant-ee) and [Paresh Mahajan](https://github.com/pareshmahajan) at [Expert Talk meetup](https://www.meetup.com/expert-talks-Pune/events/249861789/) 

This repo contains overall details of the producer and consumer services, plus required docker files to complete the setup for the demo.

# Provider service

Code for provider user service can be found [here](https://github.com/prashant-ee/user-service)

# Consumer service

Code for consumer order service can be found [here](https://github.com/prashant-ee/order-service)

# Forking the repositories

1. Fork current repo, user-service and order-service repositories. See [here](https://help.github.com/articles/fork-a-repo/) on how to fork the repositories.

2. Clone the repositories from your fork.

3. Update the pom.xml file present in the order-service and user-service repos so that it correctly points to your fork as follows.

```
<scm>
    <connection>scm|git:<point-to-your-forked-git-url></connection>
    <url>scm|git:<point-to-your-forked-git-url></url>
    <developerConnection>scm:git:<point-to-your-forked-git-url></developerConnection>
    <tag>HEAD</tag>
</scm>
```
e.g.if the forked git url is 'git@github.com:pareshmahajan/user-service.git' then replace it in the above block. Above block will look like:
        
```
<scm>
    <connection>scm|git:git@github.com:pareshmahajan/user-service.git</connection>
    <url>scm|git:git@github.com:pareshmahajan/user-service.git</url>
    <developerConnection>scm:git:git@github.com:pareshmahajan/user-service.git</developerConnection>
    <tag>HEAD</tag>
</scm>
```
4. Push the changes made in the pom.xml file in the respective order-service and user-service forked repos of your own.
 
5. Use the forked urls in the jenkins jobs as well.

# Pre-requisites before the next set up:
In this workshop, we are going to create a jenkins pipeline for couple of microservices and the pipelines for consumer driven contracts. We are going to use docker to set up everything locally and hence we will recommend to use good configuration machine to the set up (atlest 16 GB RAM, quad core processor)

1. One must have installed docker locally before starting the next set up. How to install docker locally can be found [here](https://docs.docker.com/install/).

After installing docker, change the following settings in the docker preferences -> Advanced:

4 CPUs, 6 GiB RAM, Swap Memory 1 GiB

2. If you don't have maven installed already on your machine then install it using this [link](https://maven.apache.org/install.html)
If not present, Create a directory called '~/.m2'.
Commands to verify and create directory are:
```
ls -l ~/.m2
mkdir ~/.m2
```


# Creating Jenkins image:

Build the jenkins docker image using dockerfile with following command
```
docker build -t cdc-expert-talk/jenkins-cdc -f ./jenkinsDockerfile .
```
This will create the jenkins build with the suggested plugins. The plugin list is taken from [https://github.com/jenkinsci/jenkins/blob/jenkins-2.19.4/core/src/main/resources/jenkins/install/platform-plugins.json](https://github.com/jenkinsci/jenkins/blob/jenkins-2.19.4/core/src/main/resources/jenkins/install/platform-plugins.json)


# Setting up local environment:

To run the scenarios locally, along with these git repositories, a Jenkins server is required to create and run pipeline jobs. A nexus server is required to host consumer and provider's released artifacts. A Pact broker server and Postgres SQL DB to hold the Pact contracts. 

### Make Jenkins, Pact Broker and Nexus accessible on local:

Add following entries in `/etc/hosts` to allow access to these services 

```
127.0.0.1 mynexus
127.0.0.1 broker_app
127.0.0.1 jenkins
```

### Add settings.xml:

Use the settings.xml and add it under the local machines ```~/.m2/``` directory. This settings file will enable ```user-service``` and ```order-service``` to resolve artifacts from local nexus repository.

Assuming you are at the root directory of this repo, use following command to copy settings.xml:
```
cp settings.xml ~/.m2/
```

### Test setup by running Order service tests locally:

Run tests locally with command `mvn clean install`

### Test setup by running User service (make sure to use the right command or avoid running) locally:

Run test locally with command `mvn clean install -Dpact.verifier.publishResults=false`

(The test might fail if the pact broker is not running or there are not pacts in the pact broker, it will work once you publish the pacts from the order service pacts).

### Start all the services locally:

To start all of these run the docker compose command :

```
docker-compose up
```

### Verify services are up and running locally:
Verify by hitting following urls in the browser:

Pact Broker:
http://broker_app:80

Nexus:
http://mynexus:8081/nexus

Jenkins:
http://jenkins:8080


### Setup Jenkins

#### Create appropriate maven setup on Jenkins

- Login to Jenkins container with 
```
docker exec -it <JenkinsContainerId> /bin/bash
```
- Run following commands to copy setttings.xml to Jenkins box:

```
mkdir /var/jenkins_home/.m2
mkdir /var/jenkins_home/.m2/repository
cp /tmp/settings.xml /var/jenkins_home/.m2/
```

#### Create an SSH key for Jenkins and add it in github account

Create ssh key with instructions from 
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-linux

Add ssh key to github account
https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/#platform-linux

#### Setup git configuration for jenkins

Setup user and email using the following command
```
git config --global user.name "<Your Name>"
git config --global user.email "<Your email>"
```
#### Add GitHub to known hosts

Run command
```
ssh -T git@github.com
```

#### Configure slack global configuration
Use base URL : https://cdc-expert-talk.slack.com/services/hooks/jenkins-ci/

Integration token : `<generated>`
 
#### Disable CSRF (see webhook add section).

# Demo scenarios

Consumer - Order service
Provider - User service

Order service => call GET /user/{userId} => User service response with user details.

### Scenario 1 - Consumer running (passing)

### Scenario 2 - Provider passing
 - Fail with Provider state implementation missing.

### Scenario 3 - Provider failing (fullName instead of name is used in the response).

### Scenario 4 - Take consumer and provider to production.

### Scenario 5 - Consumer Failing 
- Create webhook here first.
- Pact changes
- Start using a new field that is not provided by the provider e.g. primeMemberId.
- fix with the producer change by sending the primeMemberId details as well.

### Scenario 6 - Deploy Provider to DEV.

### Scenario 7 - Deploy Consumer to DEV - can-i-deploy pass.

### Scenario 8 - Deploy consumer to PROD - can-i-deploy fail. 
- Fix by deploying the provider to Prod and then promote consumer to PROD.

### Scenario 9 - Consumer Passing changes (Pre-verified contracts)(no contract change)
- Some internal implementation change. 

# Webhook creation

- Edit job configuration which needed to be triggred remotely. 
- Enable 'Trigger build remotely' and provide a appropriate token. 
- Note down the url provided in the description. Save the configuration.
- Try triggring the job remotely through the url
```
curl -X POST -u "jenkins_user:jenkins_password" "url"
```
- You might need to disable CSRF as per the version of jenkins 
(refer: https://issues.jenkins-ci.org/browse/JENKINS-42200 & https://support.cloudbees.com/hc/en-us/articles/219257077-CSRF-Protection-Explained)
- Once the remote url trigger works for the job. 
- Configure a webhook in the Pact Broker by clicking on the `create webhook` link on pact broken home. 
- Send the payload similar to following:
```
{
  "events": [{
    "name": "contract_content_changed"
  }],
  "request": {
    "method": "POST",
    "url": "http://jenkins:8080/job/user-service/job/user-service-contract-pipeline/build?token=user-service-contract-pipeline-webhook-token",
    "username": "jenkins_user",
    "password": "jenkins_password",
    "headers": {
      "Accept": "application/json"
    }
  }
}
```
- The output will also provide a link to test the webhook.
