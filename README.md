# serverless-api-app-for-load-testing
Below sample serverless application can be used for the lambda load testing and how to improvise the performance:
![image](https://github.com/user-attachments/assets/7fc96e20-7a87-42dc-848e-c96d824f69c5)

To demonstrate this, we need to create one API Gateway, one DyanamoDb table and a lambda to perform CRUD operation in the DynamoDb table. Whenever an API call made through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function and performs CRUD operation in DynamoDb. For the experimental purpose, only POST method is used in this exercise.

The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example to create a new item in the table:
```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "ABCD1234",
            "number": "1001"
        }
    }
}
```
How to configure?
Create Lambda Execution Role:
Create an IAM policy. Attach it with lambda execution role.

To create a policy:
1. Open IAM Service in console
2. Click on Policies on left side menu
3. Click on 'Create policy' button on right side
4. Select the "lambda" as service
5. Toggle Visual to JSON to open policy editor and copy the below policy which is needed for lambda function to write data to Dynamodb and upload logs.

```
   {
      "Version": "2024-11-07",
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
6. Click Next
7. Provide the Policy Name as 'lambda-apigateway' and save.

Create IAM Role:
1. Click on Roles on left side menu
2. Click on 'Create role' button on right side
3. Select 'AWS Service' as trusted entity type
4. Select Service as 'Lambda' & click Next
5. Select the 'lambda-apigateway' policy to attach & click Next
6. Provide Role name as 'lambda-apigateway-role' and click 'Create role'

Create Lambda Function
1. Open the Lambda service in console
2. Click on 'Create function' in lambda console
3. Select "Author from scratch". Use name 'LambdaFunctionOverHttps' , select Python 3.9 as Runtime. Under Permissions, 4. select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down
5. Click "Create function"

   ![image](https://github.com/user-attachments/assets/7d7c7a76-ae08-45de-9c1f-dff86613febe)
```
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
Test Lambda Function
Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.

Click on 'Test' & select 'Create new test event'
Provide the Event Name
Paste the following JSON into the event JSON and save
```
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```
Click Test. It will run and provide the output in console.

![image](https://github.com/user-attachments/assets/d2358ea6-8a4a-46b1-85de-cde7f5453829)

We're all set to create DynamoDB table and an API using our lambda as backend!

Create DynamoDB Table:
Create the DynamoDB table that the Lambda function uses.

To create a DynamoDB table

1.Open the DynamoDB console.
2. Choose Create table.
3. Create a table with the following settings.
    Table name – lambda-apigateway
    Primary key – id (string)
4. Choose Create.

Create API

1. Go to API Gateway console
2. Click Create API

![image](https://github.com/user-attachments/assets/8a25cf42-7d81-4ed5-9756-f439df0f3965)

3. Scroll down and select "Build" for REST API
4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"
5. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time we only have the root resource, but we need to add a resource next.
6. Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource".
7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".
8. Select "POST" from drop down , then click checkmark
9. The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function that we created earlier. As we start typing the name, our function name will show up.Select and click "Save". A popup window will come up to add resource policy to the lambda to be invoked by this API. Click "Ok"

Deploy API
We have to deploy the API which was created above to make it live.

1. Click "Actions", select "Deploy API"
2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy".
3. Now everything is set up for the solution. To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen.

Test the Solution
As API endpoints are noted, it can be invoked via multiple ways like using - Postman, Curl, Step functions etc. Below JSON message can be used to test the POST method:
```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```
Now, we have successfully created a serverless API app using API Gateway, Lambda, and DynamoDB!

Cleanup:

Delete the DynamoDB table finally after completion of all tests.
