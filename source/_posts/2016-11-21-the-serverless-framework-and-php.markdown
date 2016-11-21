---
layout: post
title: "The Serverless Framework and PHP"
date: 2016-11-21 09:21
comments: true
categories: [aws, serverless, php]
description: How to call a PHP function from within an AWS lambda using the Serverless Framework.
---
The goal of this post is to explain how to call a PHP function from within an AWS lambda using the [Serverless Framework](https://serverless.com/). 

## Prerequisites ##

First check everything we need is installed.

    $ serverless --version
    1.1.0
    $ node --version
    v7.1.0

## Install the sample PHP function ##

Install my sample _Hello_ function from my github repository.

    $ sls install --url https://github.com/ZeroSharp/serverless-php

```sh
Serverless: Downloading and installing "serverless-php"…
Serverless: Successfully installed "serverless-php".
```

## The code ##

    $ cd serverless-php

Let's have a look at the `serverless.yml` file.

```yml serverless.yml
service: serverless-php

provider:
  name: aws
  runtime: nodejs4.3
  # region: eu-west-1

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
```

Now look at the php function `index.php` that we'd like our lambda to call.

```php index.php
<?php

# $argv will contain the event object. You can output its contents like this if you like
#var_export($argv, true);

printf('Go Serverless v1.0! Your PHP function executed successfully!');
```

And the `handler.js` for the hello function looks as follows. It defines a simple lambda which calls the PHP binary, logs any errors and returns the result.

```js handler.js
'use strict';

var child_process = require('child_process');

module.exports.hello = (event, context, callback) => {

  var strToReturn = '';

  var php = './php';
  
  // workaround to get 'sls invoke local' to work
  if (typeof process.env.PWD !== "undefined") {
    php = 'php';
  }

  var proc = child_process.spawn(php, [ "index.php", JSON.stringify(event), { stdio: 'inherit' } ]);

  proc.stdout.on('data', function (data) {
    var dataStr = data.toString()
    // console.log('stdout: ' + dataStr);
    strToReturn += dataStr
  });

  // this ensures any error messages raised by the PHP function end up in the logs
  proc.stderr.on('data', function (data) {
    console.log(`stderr: ${data}`);
  });

  proc.on('close', function(code) {
    if(code !== 0) {
      return callback(new Error(`Process exited with non-zero status code ${code}`));
    }

    const response = {
      statusCode: 200,
      body: JSON.stringify({
        message: strToReturn,
        //input: event,
      }),
    };

    callback(null, response);
  });
};
```

Included is the PHP binary to bundle with our serverless function. 

(You may need to compile it yourself with different options. See below for help on how to do this.)

Check it works from your shell.

    $ php index.php

```sh
Go Serverless v1.0! Your PHP function executed successfully!
```

Run it locally through the Serverless Framework.

    $ sls invoke local --function hello

```sh
Serverless: Your function ran successfully.

{
    "statusCode": 200,
    "body": "{\"message\":\"Go Serverless v1.0! Your PHP function executed successfully!\"}"
}
```

Looks good. Let's deploy.

    $ sls deploy

```sh
Serverless: Packaging service…
Serverless: Uploading CloudFormation file to S3…
Serverless: Uploading service .zip file to S3…
Serverless: Updating Stack…
Serverless: Checking Stack update progress…
..........
Serverless: Stack update finished…

Service Information
service: serverless-php
stage: dev
region: eu-west-1
api keys:
  None
endpoints:
  GET - https://c1w0hct166.execute-api.eu-west-1.amazonaws.com/dev/hello
functions:
  serverless-php-dev-hello: arn:aws:lambda:eu-west-1:962613113552:function:serverless-php-dev-hello
```

Run the remote function via Serverless.

    $ sls invoke --function hello

```sh
{
    "statusCode": 200,
    "body": "{\"message\":\"Go Serverless v1.0! Your PHP function executed successfully!\",\"input\":{}}"
}
```

Visit the endpoint in your browser.

```json
{
    "message": "Go Serverless v1.0! Your PHP function executed successfully!"
}
```

Nice. It's all working.

## Rebuilding the PHP binary ##

Depending on the PHP function you need to run, it may be necessary to rebuild the php binary with different flags and dependencies. You can do this best with docker.

    $ docker --version
    Docker version 1.12.3, build 6b644ec

Modify `dockerfile.buildphp` as necessary.

Then run:

    $ sh buildphp.sh

This will build a new PHP binary and copy it to the project root. You can immediately deploy for testing with:

    $ sls deploy

## Thanks ##

Shout out to [Danny Linden](https://github.com/dannylinden/aws-lambda-php) whose code got me started on this. 





