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