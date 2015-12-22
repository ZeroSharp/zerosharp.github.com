---
layout: post
title: "Serverless Framework - Part 1: Up and running"
date: 2015-12-22 20:26
comments: true
categories: [aws, serverless]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply.
---
I was in the middle of a blog post about the JAWS framework and before I had finished it changed its name to [the Serverless framework](https://github.com/serverless/serverless). It is a very clever way to build apps without worrying about provisioning server or whether it will scale. This is because it uses Amazon Web Services and in particular the Amazon lambda compute service. It's currently in beta.

Follow [the instructions](http://docs.serverless.com/docs/configuring-aws) for setting up an administrative IAM user for use with the framework.

Make sure you have node and npm installed. You need node 4.0 or greater.

    $ node -v
    v4.2.3
    $ npm -v
    2.14.7

Install the Serverless framework.

    $ npm install serverless -g 
    
Create a new project

    $ serverless project create

```
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                         serverless.com, v.0.0.15
`-------'
Serverless: Enter a project name:  (serverlessVyxSv1W8e) potdCheck
Serverless: Enter a project domain (used for serverless regional bucket names):  (myapp.com) serverless.zerosharp.com
Serverless: Enter an email to use for AWS alarms:  (me@myapp.com) potdCheck@nosredna.com
Serverless: Select a region for your project: 
    us-east-1
    us-west-2
  > eu-west-1
    ap-northeast-1
Serverless: Select an AWS profile for your project: 
  > default
Serverless: Creating a project region bucket on S3: serverless.euwest1.zerosharp.com...  
Serverless: Creating CloudFormation Stack for your new project (~5 mins)...  
Serverless: Successfully created project: serverlessPotdCheck
```

It takes about 5 minutes to setup the necessary CloudFormation stack for your project. Change directory to the newly created project.

    $ cd serverlessPotdCheck
    
Create a new module.

    $ serverless module create
    
```
Serverless: Enter a name for your new module:  potd
Serverless: Enter a function name for your new module:  check
Serverless: Successfully created function: "check"  
Serverless: Installing "serverless-helpers" for this module via NPM...  
serverless-helpers-js@0.0.3 node_modules/serverless-helpers-js
└── dotenv@1.2.0
Serverless: Successfully created new serverless module "potd" with its first function "check"  
```   
 
This has created the javascript code for a basic lambda function which we can immediately deploy.

    $ serverless function deploy
    
```    
Serverless: Deploying functions in "development" to the following regions: eu-west-1  
Serverless: | 123123123 { Code: 
   { S3Bucket: 'serverless.euwest1.zerosharp.com',
     S3Key: 'Serverless/serverlessPotdCheck/development/lambdas/check@1450717964169.zip' },
  FunctionName: 'serverlessPotdCheck-check',
  Handler: 'modules/potd/check/handler.handler',
  Role: 'arn:aws:iam::962613113552:role/serverlessPotdCheck-development-r-IamRoleLambda-12LOSLPN3JHS8',
  Runtime: 'nodejs',
  Description: 'Serverless Lambda function for project: serverlessPotdCheck',
  MemorySize: 1024,
  Publish: true,
  
  ...
  
Serverless: Successfully deployed functions in "development" to the following regions: eu-west-1  
```

Now deploy the endpoints.
    
    $ serverless endpoint deploy

```
Serverless: Deploying endpoints in "development" to the following regions: eu-west-1  
Serverless: Successfully deployed endpoints in "development" to the following regions:  
Serverless: eu-west-1 ------------------------  
Serverless:   GET - https://udjkzpj2a2.execute-api.eu-west-1.amazonaws.com/development/potd/check
```

Now open a browser and navigate to the URL in the last line. You should see the following JSON response.

```json
{
    message: "Your Serverless function ran successfully!"
}
``` 

Now that's already fantastic. With a handful of commands we have deployed an arbitrary javascript function to a URL endpoint very cheaply and with automatic scaling. We never had to consider instance size or memory or operating system. 

And it's extremely extensible too - thanks to the other AWS services, we can easily make it secure (with Amazon Cognito) kick off emails (SES), store files (S3), add persistence (DynamoDB), etc. 

In the next post, I'll be applying it to a real life scenario to replace an existing web service.
