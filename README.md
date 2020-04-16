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

## Workflow

We want to use an s3 upload event that will trigger our lambda function which will resize the uploaded image and then uploads the resized image into another s3 bucket.

### Preparation

Login into your aws console, or [create a free account](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc), if you do not have yet an account.

The lambda function needs permission to access s3 and write its logs into CloudWatch, so we create first a role for this purpose and assign our function to it later.

#### Execution role

Go to IAM, Roles, Create Role with the name 'lambda-execution-role', then create an inline policy 'lambda-execution-policy' and attach it to the role. the policy have the following content:

´´´json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "logs:*"
            ],
            "Resource": "arn:aws:logs:*:*:*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::*",
            "Effect": "Allow"
        }
    ]
}
´´´

#### S3 buckets

Go to S3 and create a bucket with name images-XXXXX , replace the Xs with a number of your choice. copy this name and create another S3 bucket with the name images-XXXXX-resized.

Upload an image of your choice to images-XXXXX  bucket with a high resolution for example 1280x1280.

### Create the function

Go to Lambda and create function with the name `Create Thumbnail` and python 3.7 and the existing execution role we created before, then add a trigger for S3 and enter images-XXXXX  as the bucket name, after that edit the code part and paste the following code, remember again to select python 3.7 and `CreateThumbnail.handler` as handler:

```python
import boto3
import os
import sys
import uuid
from PIL import Image
import PIL.Image

s3_client = boto3.client('s3')

def resize_image(image_path, resized_path):
    with Image.open(image_path) as image:
        image.thumbnail((128, 128))
        image.save(resized_path)

def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        download_path = '/tmp/{}{}'.format(uuid.uuid4(), key)
        upload_path = '/tmp/resized-{}'.format(key)

        s3_client.download_file(bucket, key, download_path)
        resize_image(download_path, upload_path)
        s3_client.upload_file(upload_path, '{}-resized'.format(bucket), key)

```

Hit Save. Now we could Test the function, but first we need a test template, click Test and choose S3 Put request, and name it Upload, the template will be shown, change the name of the bucket with your bucket name: images-XXXXX , there two of them. Then change the object name `test/key` with the filename you uploaded before. Now you could click Test and you will see the result, which should be success. If you now visit the S3 bucket with images-XXXXX-resized name, you see the resized version of your image.

You could now upload another hight resolution image and trigger your lambda function.

