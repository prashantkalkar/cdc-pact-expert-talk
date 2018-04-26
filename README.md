# cdc-pact-expert-talk
Expert talk repo to contain overall details of the producer and consumer services, plus required docker files to complete the setup for the demo.

# Provider service

Code for provider user service can be found [here](https://github.com/prashant-ee/user-service)

# Consumer service

Code for consumer order service can be found [here](https://github.com/prashant-ee/order-service)

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
