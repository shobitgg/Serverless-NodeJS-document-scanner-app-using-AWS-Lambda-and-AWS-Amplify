
# 
Project:
https://github.com/amazon-archives/aws-serverless-document-scanner
- https://aws.amazon.com/blogs/compute/building-a-serverless-document-scanner-using-amazon-textract-and-aws-amplify/
- https://www.youtube.com/watch?v=kA2ZYD4zgEo

# Building a serverless document scanner using Amazon Textract and AWS Amplify
This guide demonstrates creating and deploying a production ready document scanning application. It allows users to manage projects, upload images, and generate a PDF from detected text. The sample can be used as a template for building expense tracking applications, handling forms and legal documents, or for digitizing books and notes.

The frontend application is written in Vue.js and uses the Amplify Framework. The backend is built using AWS serverless technologies and consists of an Amazon API Gateway REST API that invokes AWS Lambda functions. Amazon Textract is used to analyze text from uploaded images to an Amazon S3 bucket. Detected text is stored in Amazon DynamoDB.

![image](https://user-images.githubusercontent.com/41900814/236997474-2a021b37-4004-45e5-b3b1-1ac04ad6ac71.png)


Prerequisites
You need the following to complete the project:

Node.js and npm installed on a computer.
An AWS account. This project can be completed using the AWS Free Tier.

Deploy the application

![image](https://user-images.githubusercontent.com/41900814/236997525-29ea6cb1-d718-45c9-9c94-3ec80263f649.png)


The solution consists of two parts, the frontend application and the serverless backend. The Amplify CLI deploys all the Amazon Cognito authentication, and hosting resources for the frontend. The backend requires the Amazon Cognito user pool identifier to configure an authorizer on the API. This enables an authorization workflow, as shown in the following image.


![image](https://user-images.githubusercontent.com/41900814/236997541-ae451952-47d8-4ca7-a767-e08cca28bcb2.png)


First, configure the frontend. Complete the following steps using a terminal running on a computer or by using the AWS Cloud9 IDE. If using AWS Cloud9, create an instance using the default options.

From the terminal:

Install the Amplify CLI by running this command.
```
npm install -g @aws-amplify/cli
```
Configure the Amplify CLI using this command. Follow the guided process to completion.
```
amplify configure
```
Clone the project from GitHub.
```
git clone https://github.com/aws-samples/aws-serverless-document-scanner.git
```
Navigate to the amplify-frontend directory and initialize the project using the Amplify CLI command. Follow the guided process to completion.
```
cd aws-serverless-document-scanner/amplify-frontend

amplify init
```
Deploy all the frontend resources to the AWS Cloud using the Amplify CLI command.
```
amplify push
```
After the resources have finishing deploying, make note of the StackName and UserPoolId properties in the amplify-frontend/amplify/backend/amplify-meta.json file. These are required when deploying the serverless backend.


![image](https://user-images.githubusercontent.com/41900814/236997701-a5e5ae01-32b4-489a-9d3c-d907c0cd130a.png)


1. Next, deploy the serverless backend. While it can be deployed using the AWS SAM CLI, you can also deploy from the AWS Management Console:

2. Navigate to the document-scanner application in the AWS Serverless Application Repository.
In Application settings, name the application and provide the StackName and UserPoolId from the frontend application for the UserPoolID and AmplifyStackName parameters. Provide a unique name for the BucketName parameter.

![image](https://user-images.githubusercontent.com/41900814/236997759-3649e2ec-049a-4f7b-afcc-635b10ca5832.png)
3. Choose Deploy.
4. Once complete, copy the API endpoint so that it can be configured on the frontend application in the next section.

![image](https://user-images.githubusercontent.com/41900814/236997839-e2b02462-f801-4b81-a43f-5f2bd8d58b62.png)

# Configure and run the frontend application
1. Create a file, amplify-frontend/src/api-config.js, in the frontend application with the following content. Include the API endpoint and the unique BucketName from the previous step. The s3_region value must be the same as the Region where your serverless backend is deployed.

```
const apiConfig = {
	"endpoint": "<API ENDPOINT>",
	"s3_bucket_name": "<BucketName>",
	"s3_region": "<Bucket Region>"
};

export default apiConfig;

```

2. In a terminal, navigate to the root directory of the frontend application and run it locally for testing

```
cd aws-serverless-document-scanner/amplify-frontend

npm install

npm run serve
```

You should see an output like this:

![image](https://user-images.githubusercontent.com/41900814/236997940-f5c1e503-7799-471b-b940-28af672d4b88.png)


3. To publish the frontend application to cloud hosting, run the following command.
```
amplify publish
```
Once complete, a URL to the hosted application is provided.

![image](https://user-images.githubusercontent.com/41900814/236997981-289ae7e4-d5ed-435d-a078-92ea5ce9814e.png)



# Using the frontend application
Once the application is running locally or hosted in the cloud, navigating to it presents a user login interface with an option to register. The registration flow requires a code sent to the provided email for verification. Once verified you’re presented with the main application interface.
![image](https://user-images.githubusercontent.com/41900814/236998090-1f7e870c-8c30-48a3-a21d-687977ca4b36.png)


Once you create a project and choose it from the list, you are presented with an interface for uploading images by page number.

![image](https://user-images.githubusercontent.com/41900814/236998116-47972cb1-f199-4c79-9595-f5359e25b121.png)


On mobile, it uses the device camera to capture images. On desktop, images are provided by the file system. You can replace an image and the page selector also lets you go back and change an image. The corresponding analyzed text is updated in DynamoDB as well.


![image](https://user-images.githubusercontent.com/41900814/236998143-92480ed0-fdc0-42c0-a372-0e04b2eb3623.png)



Each time you upload an image, the page is incremented. Choosing “Generate PDF” calls the endpoint for the GeneratePDF Lambda function and returns a PDF in base64 format. The download begins automatically.


![image](https://user-images.githubusercontent.com/41900814/236998185-5cadbd18-2df9-49bf-820d-6373eed4de59.png)


You can also open the PDF in another window, if viewing a preview in a desktop browser.
![image](https://user-images.githubusercontent.com/41900814/236998214-fa4e926d-872f-46ff-883d-7e8fc5ac1557.png)


Understanding the serverless backend

![image](https://user-images.githubusercontent.com/41900814/236998241-1c64e51d-e702-4cd3-9018-ed545c3c9f8a.png)


In the GitHub project, the folder serverless-backend/ contains the AWS SAM template file and the Lambda functions. It creates an API Gateway endpoint, six Lambda functions, an S3 bucket, and two DynamoDB tables. The template also defines an Amazon Cognito authorizer for the API using the UserPoolID passed in as a parameter:

This only allows authenticated users of the frontend application to make requests with a JWT token containing their user name and email. The backend uses that information to fetch and store data in DynamoDB that corresponds to the user making the request.

Two DynamoDB tables are created. A Project table, which tracks all the project names by user, and a Pages table, which tracks pages by project and user. The DynamoDB tables are created by the AWS SAM template with the partition key and range key defined for each table. These are used by the Lambda functions to query and sort items. See the documentation to learn more about DynamoDB table key schema.
```
ProjectsTable:
    Type: AWS::DynamoDB::Table
    Properties: 
      AttributeDefinitions: 
        - 
          AttributeName: "username"
          AttributeType: "S"
        - 
          AttributeName: "project_name"
          AttributeType: "S"
      KeySchema: 
        - AttributeName: username
          KeyType: HASH
        - AttributeName: project_name
          KeyType: RANGE
      ProvisionedThroughput: 
        ReadCapacityUnits: "5"
        WriteCapacityUnits: "5"

  PagesTable:
    Type: AWS::DynamoDB::Table
    Properties: 
      AttributeDefinitions: 
        - 
          AttributeName: "project"
          AttributeType: "S"
        - 
          AttributeName: "page"
          AttributeType: "N"
      KeySchema: 
        - AttributeName: project
          KeyType: HASH
        - AttributeName: page
          KeyType: RANGE
      ProvisionedThroughput: 
        ReadCapacityUnits: "5"
        WriteCapacityUnits: "5"
YAML
```

