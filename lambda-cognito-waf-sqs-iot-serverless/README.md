# Amazon WAF to Amazon API Gateway to Amazon Cognito to Amazon SQS to AWS Lambda to Amazon DynamoDB to AWS IOT 

This pattern explains how to deploy a SAM application with Amazon WAF, Amazon Cognito, Amazon API Gateway, Amazon SQS, AWS Lambda, Amazon DynamoDB and AWS IOT

This pattern is useful to accept and respond to requests quickly but offloading the processing as asynchronous process. Once the request is placed in SQS, API gateway responds back to the caller immediately without waiting for those messages to be processed.

When an HTTP POST request is made to the Amazon API Gateway endpoint, request goes through the firewall rules in AWS WAF then Gateway authorizes the request with Cognito Authorizer (OAuth flow) against the authentication header in request and OAuth Scope. When credentials are valid, request payload is sent to Amazon Simple Queue Service. AWS Lambda function consumes event from the Queue and looks up in a Amazon DynamoDB table to the IoT Topic and post the event/payload as MQTT message to AWS IoT Topic. 

Key Benefits:

Operations: AWS services used in this pattern can scale easily and quickly based on incoming traffic.
Security: The APIs created with Amazon API Gateway expose HTTPS endpoints only. APIs are secured with OAuth2.0 flow using Cognito Authorizer. All user data stored in Amazon DynamoDB and Amazon SQS is fully encrypted at rest. 
Reliability: A dead-letter queue ensures no messages are lost due to issues. Helps debugging failed messages and once underlying issue is resolved, messages can be redrived back to source queues for processing.
Performance: AWS Lambda supports batching with Amazon SQS. Lambda reads messages in batches and invokes the function once for each batch.
Cost: Pay only for what you use. No minimum fee.
Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the AWS Pricing page for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Requirements

* [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and log in. The IAM user that you use must have sufficient permissions to make necessary AWS service calls and manage AWS resources.
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) (AWS SAM) installed

## Deployment Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ``` 
    git clone https://github.com/aws-samples/serverless-patterns
    ```
1. Change directory to the pattern directory:
    ```
    cd _patterns-model
    ```
1. From the command line, use AWS SAM to deploy the AWS resources for the pattern as specified in the template.yml file:
    ```
    sam deploy --guided
    ```
1. During the prompts:
    * Enter a stack name
    * Enter the desired AWS Region
    * Allow SAM CLI to create IAM roles with the required permissions.

    Once you have run `sam deploy --guided` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy` in future to use these defaults.

1. Note the outputs from the SAM deployment process. These contain the resource names and/or ARNs which are used for testing.

## How it works

Explain how the service interaction works.

## Testing

1. Add a DynamoDB recrod with ClientID = 'DemoClient' and Topic = "MyTopic"

2. Go to AWS IOT Core Console -> Create MQTT Test Client -> Subscribe to a topic "MyTopic"

3. Exchange Client credential for an access token (Bearer, JWT)
curl -X POST --user 146l86s6rome5142av8csbd2ea:c5o4cqu4etftvt7kp17e5lu0m9v16fhv9r5vtcitn74ijkffmgu 'https://sibesg.auth.ap-south-1.amazoncognito.com/oauth2/token?grant_type=client_credentials&scope=sibesg/sendqrcode' -H 'Content-Type: application/x-www-form-urlencoded'

4. Copy the API gateway endpoint to send request
    ex: https://********.amazonaws.com/stag/postmessagetosqs

5. Create a JSON Request 
    ex: {
            "ClientID": "DemoClient",
            "QRCode": "vwWT)Np=+6-vGzTiopED",
            "Time": "20231201T231010",
            "STATE": "ACTIVE",
            "LOCATION": "LT100LN200",
            "TEST": "demo message 26-09 - message 1"
        }

6. Make a HTTP post request to the endpoint captured in Step 4 with Header Key = "Authorization" and value = <Access Token captured in Step 3>

7. You should get a 200 Success SendMessageResponse. You can check the MQTT Test client showing the request you passed to a dynamic client

## Cleanup
 
1. Delete the stack
    ```bash
    aws cloudformation delete-stack --stack-name STACK_NAME
    ```
1. Confirm the stack has been deleted
    ```bash
    aws cloudformation list-stacks --query "StackSummaries[?contains(StackName,'STACK_NAME')].StackStatus"
    ```
----
Copyright 2023 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0