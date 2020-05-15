# AWS Lambda hands-on

This hands-on will describe the aws lambda functions and you will create your first lambda function and examine the capabilities of lambda service.

## What is AWS Lambda

AWS Lambda is a compute service that runs your code in response to events and automatically manages the compute resources for you, making it easy to build applications that respond quickly to new information.

AWS Lambda starts running your code within milliseconds of an event such as an image upload, in-app activity, website click, or output from a connected device.

You can also use AWS Lambda to create new back-end services where compute resources are automatically triggered based on custom requests.

Workflows that may consider to use Lambda:

- Backend processing
- Event Processing
- Stream Processing
- Data Processing

AWS resources such s3 events, dynamo db operations, message queueing. AWS Lambda could be setup to service API endpoints, that would allow users to make service calls that would trigger lambda functions.

## Resize image

Using S3 and lambda function we [resize an image](resize-image) after upload and save it into a S3 Bucket.

## Text to Speech

[Serverless Application](text-to-speech) that converts a text into speech with the selected voice language.
