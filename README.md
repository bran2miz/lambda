# Lambda Project

## Author
Brandon Mizutani

## Version
1.0.0

## Overview
This project demonstrates the implementation of Lambda functions triggered by API Gateway and S3 events. The Lambda functions facilitate user authentication via Auth0 and updating of image metadata stored in S3 buckets. 

## Functionality
- User logs in via Auth0 and receives an email notification.
- API Gateway triggers a Lambda function which retrieves the relevant user record.
- Client Lambda function interacts with S3, facilitating file uploads and updates to image metadata.
- Owner Lambda function is granted access to owner S3 bucket for necessary operations.

## Project Structure
- **API Gateway** - Facilitates communication between client and serverless functions.
- **Client Lambda** - Handles user authentication and interaction with S3.
- **Owner Lambda** - Manages S3 bucket operations for authorized users.

## Usage
1. Ensure proper setup of AWS S3 bucket permissions following provided directions.
2. Deploy Lambda functions using AWS Lambda console.
3. Test Lambda functions using provided test events.
4. Verify functionality by uploading files to S3 bucket and observing image metadata updates.

## AWS S3 Setup
Follow the below steps to set up AWS S3 bucket permissions:
1. Go to AWS console.
2. Navigate to S3.
3. Create a new bucket.
4. Deselect "Block all public access" and acknowledge the settings.
5. Set permissions with the provided bucket policy from [this link](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html).
6. Save changes.

## Lambda Function Configuration
Follow these steps to configure Lambda functions:
1. Create a new Lambda function with Node.js 16.x runtime.
2. Test the function using provided test events.
3. Modify the function code as required.
4. Deploy the function.

## Lambda Trigger Configuration
To configure Lambda triggers:
1. Click on trigger.
2. Select S3 trigger.
3. Choose the Lambda file to be triggered.
4. Configure the event as necessary.

## Images.json File
The images.json file contains an array of objects representing uploaded images. Each object follows the structure:

{ "name": "imageName", "type": "jpg", "size": 1460 }

Ensure the file is updated based on new image uploads.

## Additional Information
Link to images.json with sample image uploads: [images.json](https://bran2miz-lambda.s3.us-west-2.amazonaws.com/images.json)


# Notes

user logs in -----> gets email from AuthO

use apiGateway to trigger a lambda - serverless function (aka run this chunk of code hosted in the cloud)

sends back the relevant user record

        API Gateway      
        -------->             ------------->
client                lambda                  S3
       <---------            <---------------

owner lambda will have access to the owner S3 

Lab 17 Notes:

1. Upload a new image to our s3 bucket
2. the s3 bucket should contain a dictionary of the images in the bucket
    - any time we add a new image the dictionary should be updated 
3. modify the json file and put it back into the s3

Steps:
    - succesfully get a file from S3 [X]
    - trigger an event based on a file being added []

structure of the image.json file is an array of objects, each representing an image. 

shape of each object:
{name: "imageName", type: "jpg", size: 1460}

AWS S3 directions:

- Go to AWS console 
- Go to S3
  - Create bucket
  - Deselect Block all public access
  - Check off the I acknowledge that the current settings.
  - create bucket
  - Set permissions with a bucket policy

Go to this link:
https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html

copy this:
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Principal": "*",
			"Effect": "Allow",
			"Resource": "arn:aws:s3:::bran2miz-lambda/*",
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			]
		}
	]
}

then save changes.

Then go to lambda
    - create function
      - name function and select NOde.js 16.x
      - create function
    - Test function
      - click on blue test button
      - Test event action should be Create new event
      - add Event name
      - Edit the Event JSON to this:
            {
                "key1": 1,
                "key2": 2,
            }
      - Add console.log(event) to console the event
      - Note: anytime you make a change for your function, you have to hit deploy
        exports.handler = async (event) => {
            console.log(event);
            const total = event.key1 + event.key2;
            // TODO implement
            const response = {
              statusCode: 200,
              body: total,
            };
            return response;
          };
        - expected value for key 'body' = 3
      - What if event is getting something to be added to S3
        - can create a new object called putObject in configure event (test drop down)
        - can select template s3-put to do an update
          - do this, deploy and test:
            exports.handler = async (event) => {
            console.log("this is the event argument", event.Records[0].s3.object.key,event.Records[0].s3.object.size);
            // TODO implement
            const response = {
                                statusCode: 200,
                                body: "hello from the lambda",
                             };
            return response;
            };
          - get the key name and size.
            - comment that out
        - add AWS library
            const AWS = require('aws-stk');
        - const S3 = new AWS.S3(); (to get the new instance of the S3 from AWS library)
        - create a test.json file that has an object (ie: [{"key":1, "key2":2}] )
          - upload it to the S3 Bucket
          - then add in Lambda:
            - let Bucket = "bran2miz-lambda";
            - let Key = "test.jspm";
        - Retreive object using this code and .promise():
            let testRecord = await S3.getObject({Bucket, Key}).promise();
            - pass in Bucket and Key 
            - What you should get back is a buffer string (small secure way to package up data)

Lambda Trigger:
    - click on trigger
      - S3 trigger
      - gonna be whatever lambda file you're using
      - trigger a POST event

NOTE: use autofill for the name of the bucket

Update the images.json file based on the new added image. 

Link to images.json with 3 image uploads
https://bran2miz-lambda.s3.us-west-2.amazonaws.com/images.json
