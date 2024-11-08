# serverless-app-load-testing
Below sample serverless application can be used for the lambda load testing and how to improvise the performance:
![image](https://github.com/user-attachments/assets/7fc96e20-7a87-42dc-848e-c96d824f69c5)

To demonstrate this, we need to create one API Gateway, one DyanamoDb table and a lambda to perform CRUD operation in the DynamoDb table. Whenever an API call made through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function and performs CRUD operation in DynamoDb. For the experimental purpose, only POST method is used in this exercise.

The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:
