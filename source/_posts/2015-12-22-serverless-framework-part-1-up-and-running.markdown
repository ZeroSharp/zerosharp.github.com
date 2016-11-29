---
layout: post
title: "Serverless Framework - Part 1: Up and running"
date: 2015-12-22 20:26
comments: true
categories: [aws, lambda, serverless]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply.
---
## New version 0.3.1 ##
{% highlight Edit: since the original version of this post, a new version 0.3.1 of Serverless was released. I have updated the tutorial below to reflect the newer version. %}

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
|       |                           serverless.com, v0.3.1
`-------'

Serverless: Initializing Serverless Project...  
Serverless: Enter a name for this project:  (serverless-vyedql) serverlessPotd
Serverless: Enter a universally unique project bucket name:  (serverless-vyedql-4jboce.com) potd-zerosharp.com
Serverless: Enter an email to use for AWS alarms:  (me@serverless-vyedql.com) potd@nosredna.com
Serverless: Select a region for your project: 
    us-east-1
    us-west-2
  > eu-west-1
    ap-northeast-1
Serverless: Select an AWS profile for your project: 
  > default
Serverless: Creating stage "dev"...  
Serverless: Creating region "eu-west-1" in stage "dev"...  
Serverless: Creating your project bucket on S3: serverless.eu-west-1.potd-zerosharp.com...  
Serverless: Deploying resources to stage "dev" in region "eu-west-1" via Cloudformation (~3 minutes)...  
Serverless: Successfully deployed "dev" resources to "eu-west-1"  
Serverless: Successfully created region "eu-west-1" within stage "dev"  
Serverless: Successfully created stage "dev"  
Serverless: Successfully initialized project "serverlessPotd"  
```

It takes about 3 minutes to setup the necessary CloudFormation stack for your project. Change directory to the newly created project.

    $ cd serverlessPotd
    
Create a new component.

    $ serverless component create
    
```
Serverless: Enter a name for your new component:  (nodejscomponent) 
Serverless: Enter a name for your component's first module:  (resource) potd
Serverless: Enter a name for your module's first function:  (show) check
Serverless: Successfully created function: "check"  
Serverless: Successfully created new serverless module "potd" inside the component "nodejscomponent"  
Serverless: Installing "serverless-helpers" for this component via NPM...  
Serverless: -----------------  
serverless-helpers-js@0.0.3 node_modules/serverless-helpers-js
└── dotenv@1.2.0
Serverless: -----------------  
Serverless: Successfully created new serverless component: nodejscomponent  
```   
 
This has created the javascript code for a basic lambda function which we can immediately deploy.

    $ serverless dash deploy
    
At the prompt select both the function and the endpoint and then select _Deploy_.    
    
```
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                           serverless.com, v0.3.1
`-------'

Use the <up>, <down>, <pageup>, <pagedown>, <home>, and <end> keys to navigate.
Press <enter> to select/deselect, or <space> to select/deselect and move down.
Press <ctrl> + <enter> to immediately deploy selected.


Serverless: Select the assets you wish to deploy:
    nodejscomponent - potd - check
      function - nodejscomponent/potd/check
      endpoint - nodejscomponent/potd/check@potd/check~GET
    - - - - -
  > Deploy

Serverless: Deploying functions in "dev" to the following regions: eu-west-1  
Serverless: ------------------------  
Serverless: Successfully deployed functions in "dev" to the following regions:   
Serverless: eu-west-1 ------------------------  
Serverless:   nodejscomponent/potd/check: arn:aws:lambda:eu-west-1:962613113552:function:serverlessPotd-nodejscomponent-potd-check:dev  

Serverless: Deploying endpoints in "dev" to the following regions: eu-west-1  
Serverless: Successfully deployed endpoints in "dev" to the following regions:  
Serverless: eu-west-1 ------------------------  
Serverless:   GET - potd/check - https://rhnjv4ms2b.execute-api.eu-west-1.amazonaws.com/dev/potd/check  
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
