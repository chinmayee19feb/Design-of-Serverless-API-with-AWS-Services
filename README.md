# Build a Serverless RESTful API with AWS API Gateway, Lambda, and DynamoDB
![AWS Serverless API drawio101](https://github.com/user-attachments/assets/15921382-5339-4767-bf4a-56db88e7c1ac)

## Overview
This Project demonstrates how to create a fully serverless RESTful API using AWS API Gateway, AWS Lambda, and Amazon DynamoDB.
The API provides a single POST endpoint that supports various DynamoDB operations, such as creating, reading, updating, deleting, and scanning items in a DynamoDB table. Additionally, it includes utility operations for testing and debugging.

## Features
- **Single POST Endpoint**: Supports multiple DynamoDB operations.
- **CRUD Operations**: Create, Read, Update, and Delete items in a DynamoDB table.
- **Utility Operations**: Includes `echo` and `ping` for testing.
- **Built-in Monitoring**: Integrated with Amazon CloudWatch for logging and monitoring.

## DynamoDB Operations Supported:
- **Create, Update, and Delete Items**: Perform CRUD operations on items in a DynamoDB table.
- **Read an Item**: Retrieve a specific item by its primary key.
- **Scan Items**: Fetch all items from the DynamoDB table.
- **List All Items**: Retrieve a list of all items in the table.

## Utility Operations:
- **Echo**: Returns the input payload for testing purposes.
- **Ping**: A simple health check to verify the API is operational.

## Built-in Monitoring and Logging
- **Leverage Amazon CloudWatch** : For logging, monitoring, and troubleshooting the API and Lambda function.

## How It Works:
- **API Gateway**: Acts as the interface for clients to interact with the backend services. It routes incoming HTTPS requests to the appropriate Lambda function.
- **Lambda Function**: Processes the incoming requests, performs the specified DynamoDB operation, and returns the result to the client.
- **DynamoDB**: Stores the data and handles the database operations triggered by the Lambda function.

## The request payload Users send in the POST request identifies the DynamoDB operation and provides necessary data. 

## Setup
- **Create Lambda IAM Role** :
    - Open IAM Console
        - **Go to the IAM section in the AWS Management Console.**
        - **Navigate to Policies.**
- **Create a Policy** :
    - Create a custom policy to grant permissions for DynamoDB and CloudWatch Logs. This policy will allow the Lambda function to:
        - **Write data to DynamoDB.**
        - **Upload logs to CloudWatch.**
- **Below is the JSON for the IAM policy:**
  ```json
    {
    "Version": "2012-10-17",
    "Statement": [
    {
      "Sid": "Stmt1428341300017",
      "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "",
      "Resource": "*",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow"
    }
    ]
    }
    ```

  - **Open IAM -> Roles to create a new role** :

      - **Open IAM Console**
      - **Go to the IAM section in the AWS Management Console.**
      - **Navigate to Roles**
  - **Create a New Role**
      - **Click on Create role -> Under Trusted entity type -> select AWS service -> choose Lambda**
      - **Name the Role : Lambda-APIGW-Role**
      - **Attach above policy**

### Create Lambda Function
**To create the function**
## 1. Click "Create function" in AWS Lambda Console
![103-Lambda-creation](https://github.com/user-attachments/assets/24188eb5-6e6e-45c9-a2e3-a69f74ae3096)

## 2. Select "Author from Scratch"

- **Give Function name as : Lambda-Function-On-HTTPs**
 
- **Select Runtime language as "Python 3.13"**

- **Architecture as x86_64**

Click "Create function" 
![104-create-lambda-function](https://github.com/user-attachments/assets/7461e5b6-555c-46ce-a988-ef98aba4d17c)
## 3. Click on the Lambda function and replace the boilerplate coding with below code
**Python Code**
```python
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```
Click on Deploy to Deploy the Code
![107-successfully-deployed](https://github.com/user-attachments/assets/187f0886-f9b2-4520-be68-ac0a934554a1)


### Test Lambda Function

Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.
1. Click the arrow on "Select a test event" and click "Configure test events"
2. test the code here 

![108](https://github.com/user-attachments/assets/304355c3-bc54-4bc6-a460-6a4a04ad339e)

3.Save the test 
![test](https://github.com/user-attachments/assets/2120ed70-2e2b-4345-84eb-13cae4cad0b9)

4.We can see the Test Output Status : Succeeded
![109-test-output](https://github.com/user-attachments/assets/2739c3be-2db9-4df6-a792-1cb1710a6aa3)


### Everything is set up to create the DynamoDB table and API with Lambda handling the backend logic.

## Create DynamoDB Table

Create the DynamoDB table that the Lambda function uses.

### To create a DynamoDB table
- **Open the DynamoDB console.**
- **Choose Create table.**
- **Create a table with the following settings.**
  - **Table name –  DynamoDB-for-Lambda-APIGW**
  - **Primary key – id (string)**

![110-dynamodb](https://github.com/user-attachments/assets/2b4e2bf8-7269-45c8-ba9d-b04355f7711e)
![111-dynamodb2](https://github.com/user-attachments/assets/1c3d4551-327d-44d3-948e-b54dea7ad7b0)

## Create API

- **To create the API
      - **Go to API Gateway console**
      - **Click Create API**

  ![111-dynamodb2](https://github.com/user-attachments/assets/6ddfe223-1ce8-44ed-bb9b-2bd47c222dc7)

- **Give the API name as "Rest-API-for-DynamoDB-Operations", keep everything as it is, click "Create API"**
  
![113-name-API](https://github.com/user-attachments/assets/daeaf51e-d06b-4f7d-a932-1637b94e6d92)

- **An API is a collection of resources and methods, integrated with backend services like HTTP endpoints, Lambda functions, or other AWS services.**
        - **click on "Create Resource"**
  ![114-create resource](https://github.com/user-attachments/assets/30a33da8-bec9-441b-b109-7385fc594102)

- **Mention resource name "DynamoDBManager" and click "Create resource"**

  ![115-nameresource](https://github.com/user-attachments/assets/039dee66-ae3e-48c8-90ba-b004463388b3)

- **create a POST method for our API. Select "/DynamoDBManager" and click "Create Method"**
  
![116-createmethod](https://github.com/user-attachments/assets/7d9c6f44-3f66-4d9e-b117-6cf60e2d7254)

- **Select**
        - **Method type: POST**
        - **Integration type: Lambda**
        - **Lambda function: Lambda-Function-On-HTTPs (which we created earlier)**
        - **Click "Create method"** 
![117-method-post](https://github.com/user-attachments/assets/ee676dc8-cfad-4d5e-b44b-d65335e644ae)
![118](https://github.com/user-attachments/assets/63811126-92fe-41b2-a74d-4028b1aa20e3)
![119](https://github.com/user-attachments/assets/f43527e2-422a-49ac-a4a5-a5d9afe61651)
Our API-Lambda integration is done
## Deploy the API
### In this step, you deploy the API that you created to a stage called Prod.
![120](https://github.com/user-attachments/assets/eba1c533-d8d7-4f95-b8eb-8f35532981b3)
 
 - **Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy**
![121-stage-DeployAPI](https://github.com/user-attachments/assets/1515c3d2-f8f9-4e7b-ad2d-e17259d67aa1)

We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

![122-invoke-url](https://github.com/user-attachments/assets/29b3ac6e-7c80-4d16-aba3-bf304273d6ae)

## Running our solution
   - **The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:**
```json
{
    "operation": "create",
    "tableName": "DynamoDB-for-Lambda-APIGW",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```
 
 - **To execute our API from local machine, we are going to use Postman.**
 - **Postman is a powerful API development platform that simplifies creating, testing, and managing APIs**
    * To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.
    ![123postman-input](https://github.com/user-attachments/assets/2357a6f5-0d9a-4fd5-b05e-9cb260bad0d1)


 - **To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "DynamoDB-for-Lambda-APIGW" table, select "Items" tab, and the newly inserted item should be displayed.**
 ![124-final-list](https://github.com/user-attachments/assets/c5dc876b-7259-4bb3-bb2c-79fe95bd455b)

 ![123postman-input](https://github.com/user-attachments/assets/bfff7e0d-f4f1-4269-b9b4-dab986750278)


 - **In Postman to get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table**

```json
{
    "operation": "list",
    "tableName": "DynamoDB-for-Lambda-APIGW",
    "payload": {
    }
}
```
![list-on postman](https://github.com/user-attachments/assets/10846b36-77a2-4929-97f1-4bcaf9df9a5c)

# 🚀 Load Testing APIs with Postman

Postman's built-in **Load Testing** feature allows you to simulate real-world traffic and analyze API performance under stress. This helps identify:

✔ **Performance bottlenecks**  
✔ **Response time degradation**  
✔ **Error rates under load**  
✔ **Scalability limits**

---

## 🛠 Configuration Guide

### ⚙️ Recommended Test Settings
| Setting | Value | Purpose |
|---------|-------|---------|
| **Timeout** | 3 sec (reduced from 5 sec) | Faster failure detection |
| **Virtual Users (VUs)** | 10 | Concurrent user simulation |
| **Memory Allocation** | 1024MB (up from 128MB) | Prevents function timeouts |
| **Reserved Concurrency** | 10 | Parallel execution slots |

---

![post-128-MB](https://github.com/user-attachments/assets/92e2604d-53c8-41a1-af9c-ba9dec575da0)


![1024-mb](https://github.com/user-attachments/assets/867280f8-782f-4c01-af07-13335d1d5625)


