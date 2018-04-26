# cdc-pact-expert-talk
Expert talk repo to contain overall details of the producer and consumer services, plus required docker files to complete the setup for the demo.

# Provider service

Code for provider user service can be found [here](https://github.com/prashant-ee/user-service)

# Consumer service

Code for consumer order service can be found [here](https://github.com/prashant-ee/order-service)

# Setting up Jenkins

Build the jenkins docker image using dockerfile with following command
```
docker build -t cdc-expert-talk/jenkins-cdc -f ./jenkinsDockerfile .
```
This will create the jenkins build with the suggested plugins. The plugin list is taken from [https://github.com/jenkinsci/jenkins/blob/jenkins-2.19.4/core/src/main/resources/jenkins/install/platform-plugins.json](https://github.com/jenkinsci/jenkins/blob/jenkins-2.19.4/core/src/main/resources/jenkins/install/platform-plugins.json)

Run the jenkins docker image with following commands:

```
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home cdc-expert-talk/jenkins-cdc
```
The volume from the host can be located here: ```/var/lib/docker/volumes``` (for mac and windows it will be inside the vms).

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

# Setup remaining

- Pact broker docker-compose along with Jenkins
- Setup nexus docker (optional but if possible greate)
- Add nexus as part of docker-compose. 
- Create required Jenkins jobs
   - Provider build job.
   - Provider deploy job (possibly can be combined).
   - Provider contract pipeline job.
   - Consumer build job.
   - Consumer deploy job. 
