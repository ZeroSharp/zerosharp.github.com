---
layout: post
title: "A concrete PHP Serverless example - export chess games in PDF"
date: 2016-11-29 17:37
comments: true
categories: [aws, lambda, serverless, php]
description: I build a serverless application for pretty-printing chess games from the lichess.org server.
---

In the last post I built a PHP capable sample project for [the Serverless Framework](https://serverless.com/). In this post, I'll show a concrete use of it.

The service I'm building connects runs a PHP function for pretty-printing chess games from the [lichess online chess server](http://lichess.org/). [James Clarke](https://github.com/clarkerubber/lichessPDFExporter) has written a PHP function to do this using [fpdf17](http://www.fpdf.org/).

The lichess exporter takes the game id of any game that has been played on the lichess server and produced a PDF output. Take for example, Game 8 of the current World Championship which is [here](https://en.lichess.org/COQChpzH). When I open the resulting file, I see this:

{% img /images/blog/serverless-lichess-pdf-exporter-001.png %}

In this blog post I'll describe how I turned this into a serverless service. The goal is to create:

* Add an endpoint which takes the game id as a parameter
* Run the PHP function via an AWS lambda function
* Return the result as a stream

## Prerequisites ##

First check everything we need is installed.

    $ serverless --version
    1.2.1
    $ node --version
    v7.1.0

## Initial setup ##

    $ mkdir serverless-lichess-to-pdf
    $ cd serverless-lichess-to-pdf
    $ sls install --url https://github.com/ZeroSharp/serverless-php

Next copy in the source from https://github.com/clarkerubber/lichessPDFExporter. 

You can check it works by running the following.

    $ php main.php COQChpzH > COQChpzH.pdf

What's going on here? The php binary (from the serverless-php project) is running main.php (from the lichess-pdf-exporter project) with argument `COQChpzH` (which corresponds to a [chess game](https://en.lichess.org/hsXtkVk8) on the lichess server. The main.php function downloads the game from the lichess API and passes it through the fpdf17 library to create a pdf stream which is written out to the `COQChpzH.pdf` file.

## Lessons learned ##

I learned a few things while trying to get this project working. The basic plan is to modify handler.js so that it return the output of the call described above. Turns out there are quite a few gotchas along the way.

## Lesson 1 - Defining a path parameter ##

I want my API to look like this:

    http://.../serverless-lichess-to-pdf/export/{gameid}

I could not find an example in the serverless docs for getting a parameter that is passed in the URL.

Turns out your `serverless.yml` file should look like this:

```yml serverless.yml
functions:
  exportToPdf:
    handler: handler.exportToPdf
    events:
      - http:
          path: export/{gameid}
          method: get
```

Then, in your handler.js you can retrieve the parameter with:

```js
module.exports.exportToPdf = (event, context, callback) => {
  var gameid = event.pathParameters.gameid; 
  // etc...
}
```  

## Lesson 2 - API Gateway does not support binary data ##

I was hoping I could just do something like this:

```js handler.js
// this does NOT work
const response = {
    statusCode: 200,
    body: outputFromPhpCall,
    content-type: "application/pdf"
};

return callback(null, response);
```

At present, you cannot return a binary file. Amazon have just (November 2016) [released support for binary types in API Gateway]( https://aws.amazon.com/blogs/compute/binary-support-for-api-integrations-with-amazon-api-gateway/) but it's currently [an open issue](https://github.com/serverless/serverless/issues/2797) in the Serverless Framework.

## Lesson 3 - You can redirect the response to an S3 bucket ##

So instead of returning the binary output, I can write the output to an S3 bucket and return a 302 redirection to the S3 resource. Like this:

```js handler.js
// body contains the output from the PHP call
const params = {
    Bucket: bucket,
    Key: key,
    ACL: 'public-read-write',
    Body: body,
    ContentType: 'application/pdf'
};

// Save the pdf file to S3    
s3.putObject(params, function(err, data) {
if (err)
{
    return callback(new Error(`Failed to put s3 object: ${err}`));
}

// respond with a 302 redirect to the PDF file
const response = {
    statusCode: 302,
    headers: {
        location : `https://s3-eu-west-1.amazonaws.com/${bucket}/${key}`
    }
};

return callback(null, response);
```

## Lesson 4 - You can automatically delete S3 objects after a number of days ##

Each S3 bucket has optional [lifecycle rules](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) where you can specify that files are automatically removed after a time period. I wanted to set this up within the `serverless.yml` resources section, but the syntax for the lifecycle rules were not very obvious and I could not find any examples online. The following seems to work:

```yml serverless.yml
resources:
  Resources:
    PackageStorage:
      Type: AWS::S3::Bucket
      Properties:
        AccessControl: PublicRead
        BucketName: ${self:custom.exportToPdfBucket}
        LifecycleConfiguration:
          Rules:
            - ExpirationInDays: 1
              Status: Enabled
```

## It's all working now ##

You can check it out by visiting [this link](https://e7tyur4sib.execute-api.eu-west-1.amazonaws.com/dev/export/COQChpzH).

The [source code is on Github](https://github.com/ZeroSharp/serverless-lichess-to-pdf).