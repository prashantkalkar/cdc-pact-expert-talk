# cdc-pact-expert-talk
Expert talk repo to contain overall details of the producer and consumer services, plus required docker files to complete the setup for the demo.

# Provider service

Code for provider user service can be found [here](https://github.com/prashant-ee/user-service)

# Consumer service

Code for consumer order service can be found [here](https://github.com/prashant-ee/order-service)

# Creating Jenkins image

Build the jenkins docker image using dockerfile with following command
```
docker build -t cdc-expert-talk/jenkins-cdc -f ./jenkinsDockerfile .
```
This will create the jenkins build with the suggested plugins. The plugin list is taken from [https://github.com/jenkinsci/jenkins/blob/jenkins-2.19.4/core/src/main/resources/jenkins/install/platform-plugins.json](https://github.com/jenkinsci/jenkins/blob/jenkins-2.19.4/core/src/main/resources/jenkins/install/platform-plugins.json)


# Setting up local environment

To run the scenarios locally, along with these git repositories, a Jenkins server is required to create and run pipeline jobs. A nexus server is required to host consumer and provider's released artifacts. A Pact broker server and Postgres SQL DB to hold the Pact contracts. 

### Make Jenkins, Pact Broker and Nexus accessible on local

Add following entries in `/etc/host` to allow access to these services 

```
127.0.0.1 mynexus
127.0.0.1 broker_app
127.0.0.1 jenkins
```

### Add settings.xml

Use the settings.xml and add it under the local machines ```~/.m2/``` directory. This settings file will enable ```user-service``` and ```order-service``` to resolve artifacts from local nexus repository.

### Test setup by running Order service tests locally

Run tests locally with command `mvn clean install`

### Test setup by running User service (make sure to use the right command or avoid running) locally.

Run test locally with command `mvn clean install -Dpact.verifier.publishResults=false`

### Start all services on local.

To start all of these run the docker compose command :

```
docker-compose up
```

### Setup Jenkins

#### Create appropriate maven setup on Jenkins

- Login to Jenkins container with 
```
docker exec -it <JenkinsContainerId> /bin/bash
```
- Go to `/var/jenkins_home` and run following commands

```
mkdir /var/jenkins_home/.m2
mkdir /var/jenkins_home/.m2/repository
cp /tmp/settings.xml .m2/
```

#### Create an SSH key for Jenkins and add it in github account

Create ssh key with instructions from 
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-linux

Add ssh key to github account
https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/#platform-linux

#### Setup git configuration for jenkins

Setup user and email using following command

git config --global user.name "<Your Name>"
git config --global user.email "<Your email>"

# Demo scenarios

Consumer - Order service
Provider - User service

Order service => call GET /user/{userId} => User service response with user details.

### Scenario 1 - Consumer running (passing)
- Create webhook after this (?)

### Scenario 2 - Provider passing
 - Fail with Provider state implementation missing.

### Scenario 3 - Provider failing (fullName instead of name is used in the response).

### Scenario 4 - Consumer Passing changes (Pre-verified contracts)(no contract change)
- Some internal implementation change. 

### Scenario 5 - Take consumer and provider to production.

### Scenario 6 - Consumer Failing 
- Pact changes
- Start using a new field that is not provided by the provider e.g. primeMemberId.
- fix with the producer change by sending the primeMemberId details as well.

### Scenario 7 - Deploy Provider to DEV.

### Scenario 8 - Deploy Consumer to DEV - can-i-deploy pass.

### Scenario 9 - Deploy consumer to PROD - can-i-deploy fail. 
- Fix by deploying the provider to Prod and then promote consumer to PROD.

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

# Setup remaining

- Add code for DEV and PROD tags in provider job. 
- Create Deploy jobs for Consumer (Prod and Dev)
- Create Deploy jobs for Provider (Prod and Dev)
- Create Provider Contract Pipeline (to verify consumer changes). 
- Do the slack integration. 
- Run the scenarios.  

# Reference
The version related to a tag of a service can be fetched from following url from pact broker
http://broker_app/pacticipants/{pacticipant}/latest-version/{tag}
