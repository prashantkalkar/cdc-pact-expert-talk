# cdc-pact-expert-talk
Expert talk repo to contain overall details of the producer and consumer services, plus required docker files to complete the setup for the demo.

# Demo scenarios

Consumer - Order service
Provider - User service

Order service => call GET /user => User service response with user details.

### Scenario 1 - Consumer running (passing)
- Create webhook after this (?)

### Scenario 2 - Provider passing

### Scenario 3 - Provider failing (userName instead of name is used in the response).

### Scenario 4 - Consumer Passing changes (Pre-verified contracts)
- Some internal implementation change. 

### Scenario 5 - Take consumer and provider to production.

### Scenario 6 - Consumer Failing 
- Pact changes
- Change field userId in the request to just Id.
- fix with the producer change to support userId and Id both.

### Scenario 7 - Deploy Provider to DEV.

### Scenario 8 - Deploy Consumer to DEV - can-i-deploy pass.

### Scenario 9 - Deploy consumer to PROD - can-i-deploy fail. 
