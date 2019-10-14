---
layout: post
title: AWS Immersion Day
---

Notes from Flatiron School engineering team's training day with AWS.

## Serverless Transformation

### AWS Serverless Foundations (Deep dive into Lambda and API Gateway)

Presenters:  
- Pranusha Manchala
- Ramesh Jetty

Why serverless? Spend more time on your app, not maintaining servers.

"No server is easier to manage than no server" #nocode

#### How does it work?

Event source => Function => Services

#### Common use cases

1. Web apps
  - static websites
  - complex apps
  - packages for Flask, Express
2. Backends
  - mobile
  - IoT
  - apps
  - services
3. Data processing
  - real time
  - MapReduce
  - Batch
4. Chatbots
5. AWS Alexa
6. Autonomous IT
  - policy engines
  - extending AWS services
  - infrastructure management

#### AWS Lambda

- Stateless
- Memory allocation: 128 MB to 30008 MB (!!) in 64 MB increments
- Function timeout 15min
- Synchronous or async
- Author with Cloud9
- Logs to CloudWatch

##### Permissions

- Execution policies
  - what can this function access?
- Function policies
  - who can invote the function?
- Resource policies
  - cross account access

### Lab: Build a dynamic, serverless web application

WildRydes

https://github.com/aws-samples/aws-serverless-workshops/tree/master/WebApplication/


### Building Security at multiple layers of the application (Cognito deep dive)

#### API Gateway

Three types of authorization:  
- Cognito User Pools => Cognito Authorizers
- Cognito Identity Pools => AWS IAM authorization
- Custom Providers => Lambda Authorizers

#### Cognito

- Cognito User Pools
  - doesn't store user info
  - supports Oauth2.0 and OpenID Connect tokens
  - supports federation
- Cognito Identity Pools
  - supports federation

### Lab: Integrate AWS Amplify with Amazon Cognito, Amazon API Gateway, AWS Lambda, and IAM to provide an integrated authentication and authorization experience

https://github.com/aws-samples/aws-serverless-workshops/tree/master/Auth

### AWS services and features you can leverage to improve the security of a serverless applications

Shared Responsibility Model of AWS: shared between AWS and customer  
- AWS responsible for security in the cloud
- Customer responsible for security outside the cloud
  - five domains listed below
  - examples: db, s3 buckes, code, etc

#### 5 domains (and services available):

- **Identity & access management**
  - Cognito
  - API Gateway
- **Infrastructure**
  - DDOS protection
    - AWS Shield
    - AWS WAF
  - Throttling/rate limiting
    - API Gateway
    - AWS Lambda
  - Network boundaries
    - API Gateway
    - AWS Lambda
- **Data**
  - Classification
    - Amazon Macie
  - Data flow
    - AWS X-Ray
  - Encryption
    - AWS KMS
    - API Gateway
    - Amazon Certificate Manager
  - Backup/replication
    - DynamoDB
    - S3
    - RDS
  - Tokenization
    - AWS Marketplace
- **Code**
  - AWS WAF
  - AWS Gateway
  - AWS Lambda
  - AWS Secrets Manager
  - Systems Manager Parameter Store
- **Logging & monitoring**
  - Logging/tracing
    - API Gateway
    - AWS Lambda
    - X-Rays
  - Metrics
    - API Gateway
    - AWS Lambda
  - Compliance validation
    - AWS Config
    - CloudWatch events
    - AWS Budgets

Vulnerability dependency check tools:  
- [OWASP](https://www.owasp.org/index.php/About_The_Open_Web_Application_Security_Project)
- Snyk
- Twistlock
- Puresec
- Protego

### Lab: Security Workshop

https://github.com/aws-samples/aws-serverless-security-workshop
