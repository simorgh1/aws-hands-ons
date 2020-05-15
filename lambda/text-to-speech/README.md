# Text to Speech Serverless Application

This serverless application provides two methods, one for sending information about new post, which should be converted into MP3 file, and one for retrieving information about the post. Both methods are exposed as RESTful web services through Amazon API Gateway.

- Static Web hosted in S3 Bucket
- Amazon API Gateway
- Lambda function for new post which sends to Amazon SNS that will trigger Lambda "Convert to Audio" function. This function uses Amazon Polly to convert the given text using the selected voice into MP3 format.
- The result is saved in a S3 bucket.
- Then a new record is stored in the dynamo db.
- The secound method, get post, retrieves the posts either by \* meaning all posts or by post id. This request is placed from the static web site hosted in S3 and invokes the Amazon Api Gateway, which triggers the Lambda GetPosts function that selects the requested post(s).

## Getting Started

### Storage and Queue

- Create DynamoDB Table posts and define the primary key as id as string.
- Create S3 buckets audioposts-{random-number}, with public access
- Create SNS Topic new_posts, copy topic arn

### Lambda functions

#### Execution Role

- Create Lambda Role _Lab-Lambda-Role_:

create _lambda-execution-role_ and edit trust relationship:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "lambda.amazonaws.com",
          "dynamodb.amazonaws.com",
          "apigateway.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

then add inline policy, lambda-execution-policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "polly:SynthesizeSpeech",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "sns:Publish",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "*"
            ],
            "Effect": "Allow"
        }
    ]
}
```

#### New Post

- Create Lambda function _PostReader_NewPost_ with python 2.7 and select existing role _Lab-Lambda-Role_
- Paste index-new-post.py code
- Set Environment variables key: _SNS_TOPIC_ value: topic arn
- Test the function with the following event:

```json
{
  "voice": "Joanna",
  "text": "This is working!"
}
```

#### Audio Converter

- Create Convert to audio lambda function. Name: _ConvertToAudio_ and python 2.7 and content of _index-convert2audio.py_ and existing role
- Set the environment vars, key: DB_TABLE_NAME and key: BUCKET_NAME
- Set the timeout to 5 min
- Add trigger SNS for topic new_posts
- Test the functions by invoking PostReader_newPost function

#### Get Post

- Create a lambda function for getting the posts. name: PostReader_GetPost, python 2.7 and using existing execution role. add the content of file index-get-post.py. Then update the environment variables with key: DB_TABLE_NAME
- Test it with the following event:

```json
{
  "postId": "*"
}
```

### Expose the Lambda function as a RESTfull Web Service

- Goto API Gateway Service and create a new public REST API: name: PostReaderAPI, endpoint type: Regional
- Then create the HTTP methods, POST and GET
- In Actions -> Create Method, POST: integration type: lambda function, name: PostReader_NewPost
- In Actions -> Create Method, GET: integration type: lambda function, name: PostReader_GetPost
- In Actions: Enable CORS (Cross-Origin Resource Sharing). -> replae existing values
- Get Method should be configured for the query parameter, postId. Click Method Request in Get Method. Expand URL Query String Parameters. Add query string, name: postId
- Then in Integration Request: Expand Mapping Templates, select, When there are no templates defined and click Add mapping template. Enter application/json and enter the following under the Generate template:

```json
{
  "postId": "$input.params('postId')"
}
```

Save and Deploy API from the Actions menu. name the stage _Dev_. Copy the invoke URL for the next step.

### Create a Serverless User Interface

- Goto web folder and update API_ENDPOINT in scripts.js with the Invoke URL you copied earlier.
- Create an S3 bucket named www-{BUCKET}, use the name of audioposts bucket, deselect the Block all public access option and update the Bucket Policy with:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::www-BUCKET/*"]
    }
  ]
}
```

- upload the content of the www folder into this bucket.
- then enable static web hosting and open the created Endpoint URL.
