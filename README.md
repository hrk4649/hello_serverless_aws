# hello_serverless_aws
a simple sample for serverless framework using AWS

## References

- https://www.serverless.com/framework/docs/providers/aws/guide/services
- https://www.serverless.com/framework/docs/getting-started

## Environment

- Windows 11
- Chocolatey 0.10.15
- nvm 1.1.7
- node 14.3.0

## What to do

- 1 setting node.js
- 2 setting AWS
- 3 setting serverless
- 4 creating a service
- 5 changing serverless.yml
- 6 deploying and accessing service
- 7 removing service

### 1 setting node.js

- install Chocolatery
- install nvm
- install node

### 2 setting AWS

- (on AWS console) create a working user
- (on AWS console) create an access key
- (on local) install aws cli v2
- (on local) configure the access key using aws cli

change region where you like

```bash
$ aws configure
AWS Access Key ID [None]: XXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXX
Default region name [None]: us-west-2
Default output format [None]:
```

### 3 setting serverless


``` npm install -g serverless ```

### 4 creating a service

create service using a template

```serverless create --template aws-nodejs --path myService```

created files

```bash
$ find myService/
myService/
myService/.npmignore
myService/handler.js
myService/serverless.yml
```

### 5 changing serverless.yml

#### 5.1 aws region

serverless doesn't look aws cli's default region.
You need to declare the region to use.

```yml
# you can overwrite defaults here
  stage: dev
  region: us-west-2
```

### 5.2 httpAPI

put "events" and "httpApi" to access the function via API Gateway

```yml
functions:
  hello:
    handler: handler.hello
    events:                       # here
      - httpApi:                  # here
          path: /users/create     # here
          method: get             # here
```

After editing serverless.yml, Checking whether serverless command can read serverless.yml correctly would be good.
Run ```serverless print```.

```bash
$ cd myService
$ serverless print
service:
  name: myservice
frameworkVersion: '2'
provider:
  stage: dev
  region: us-west-2
  name: aws
  runtime: nodejs12.x
  lambdaHashingVersion: '20201221'
  versionFunctions: true
  variableSyntax: \${([^{}:]+?(?:\(|:)(?:[^:{}][^{}]*?)?)}
functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /users/create
          method: get
    name: myservice-dev-hello
```

### 6 deploying and accessing service

let's deploy the service.

```bash
$ serverless deploy
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
........
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service myservice.zip file to S3 (569 B)...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
..............................
Serverless: Stack update finished...
Service Information
service: myservice
stage: dev
region: us-west-2
stack: myservice-dev
resources: 11
api keys:
  None
endpoints:
  GET - https://hzay6myhtb.execute-api.us-west-2.amazonaws.com/users/create
functions:
  hello: myservice-dev-hello
layers:
  None
```

You can access the endpoint.

```bash
$ curl -s https://hzay6myhtb.execute-api.us-west-2.amazonaws.com/users/create
{
  "message": "Go Serverless v1.0! Your function executed successfully!",
  "input": {
    "version": "2.0",
    "routeKey": "GET /users/create",
    "rawPath": "/users/create",
    "rawQueryString": "",
    "headers": {
      "accept": "*/*",
      "content-length": "0",
      "host": "hzay6myhtb.execute-api.us-west-2.amazonaws.com",
      "user-agent": "curl/7.73.0",
      "x-amzn-trace-id": "Root=1-61a940fe-2d0244b1668b2e442f809d44",
      "x-forwarded-for": "XXX.XXX.XXX.XXX",
      "x-forwarded-port": "443",
      "x-forwarded-proto": "https"
    },
    "requestContext": {
      "accountId": "XXXXXXXXXXX",
      "apiId": "hzay6myhtb",
      "domainName": "hzay6myhtb.execute-api.us-west-2.amazonaws.com",
      "domainPrefix": "hzay6myhtb",
      "http": {
        "method": "GET",
        "path": "/users/create",
        "protocol": "HTTP/1.1",
        "sourceIp": "XXX.XXX.XXX.XXX",
        "userAgent": "curl/7.73.0"
      },
      "requestId": "JvcXxgUBvHcEMjw=",
      "routeKey": "GET /users/create",
      "stage": "$default",
      "time": "02/Dec/2021:21:56:14 +0000",
      "timeEpoch": 1638482174359
    },
    "isBase64Encoded": false
  }
}
```

you can find the following resources were created on AWS.

- Lambda
- API Gateway to call the function
- Cloud Formation to manage resources
- S3 to store code

```bash
$ aws lambda list-functions
{
    "Functions": [
        {
            "FunctionName": "myservice-dev-hello",

-- snip 8< --

$ aws apigatewayv2 get-apis
{
    "Items": [
        {
            "ApiEndpoint": "https://hzay6myhtb.execute-api.us-west-2.amazonaws.com",

-- snip 8< --

$ aws cloudformation list-stacks
{
    "StackSummaries": [
        {
            "StackId": "arn:aws:cloudformation:us-west-2:XXXXXXXXXXX:stack/myservice-dev/2f9c2ef0-53ba-11ec-a240-0a69e1709a71",
            "StackName": "myservice-dev",
            "TemplateDescription": "The AWS CloudFormation template for this Serverless application",
            "CreationTime": "2021-12-02T21:52:47.012000+00:00",
            "LastUpdatedTime": "2021-12-02T21:53:28.737000+00:00",
            "StackStatus": "UPDATE_COMPLETE",
            "DriftInformation": {
                "StackDriftStatus": "NOT_CHECKED"
            }
        },

-- snip 8< --

$ aws s3 ls
2021-12-03 06:53:17 myservice-dev-serverlessdeploymentbucket-a7fy6sodtb0o

```

### 7 removing service

Don't forget removing the service not to pay extra charge.

```bash
$ serverless remove
Serverless: Getting all objects in S3 bucket...
Serverless: Removing objects in S3 bucket...
Serverless: Removing Stack...
Serverless: Checking Stack delete progress...
....................
Serverless: Stack delete finished...

Serverless: Stack delete finished...
```