---
layout: post
title: "Serverless Framework - Part 3: The guts of a serverless service"
date: 2016-01-29 17:44
comments: true
categories: [serverless, aws, lambda, typescript, mocha]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply. Here we discuss how a Serverless function works.
---
This is part of an ongoing series about the [Serverless framework](https://github.com/serverless/serverless). For those following along, [part 1](/serverless-framework-part-1-up-and-running/) and [part 2](/serverless-framework-part-2-typescript-and-mocha/) have been updated for the current latest version of Serverless 0.3.1.

In this post, we'll discuss how a Serverless function actually works.

## The guts of a serverless function ##

When we visited the deployed endpoint at the end of [part 1](/serverless-framework-part-1-up-and-running/), it correctly returned some JSON content.

```json
{
    message: "Your Serverless function ran successfully!"
}
``` 

Where does this message come from? Look at _index.js_ in the component's _lib_ folder.

{% codeblock lang:javascript nodejscomponent/lib/index.js %}
/**
 * Lib
 */

module.exports.respond = function(event, cb) {

  var response = {
    message: "Your Serverless function ran successfully!"
  };

  return cb(null, response);
};
{% endcodeblock %}

And it's the _handler.js_ file in the function's subfolder which calls it.

{% codeblock lang:javascript nodejscomponent/potd/check/handler.js %}
'use strict';

/**
 * Serverless Module: Lambda Handler
 * - Your lambda functions should be a thin wrapper around your own separate
 * modules, to keep your code testable, reusable and AWS independent
 * - 'serverless-helpers-js' module is required for Serverless ENV var support.  Hopefully, AWS will add ENV support to Lambda soon :)
 */

// Require Serverless ENV vars
var ServerlessHelpers = require('serverless-helpers-js').loadEnv();

// Require Logic
var lib = require('../../lib');

// Lambda Handler
module.exports.handler = function(event, context) {

  lib.respond(event, function(error, response) {
    return context.done(error, response);
  });
};
{% endcodeblock %}

In our case we're coding a password checking function. The URI will look something like this:

    http://something.amazonaws.com/development/potd/check?password=P455w0rd
    
We'll modify _lib/index.js_ to retrieve the value from the query parameter `password` and return `true` if the password is correct and `false` otherwise. But first we need to set up the parameter.

## Configuring the function parameter ##

In each function's directory, there is a file named _s-function.json_ which allows you to specify the details of the function call. Add a section to the `requestTemplates` as follows: 

{% codeblock lang:json nodejscomponent/potd/check/s-function.js %}
{
  "name": "check",
  "handler": "potd/check/handler.handler",
  "timeout": 6,
  "memorySize": 1024,
  "custom": {
    "excludePatterns": [],
    "envVars": []
  },
  "endpoints": [
    {
      "path": "potd/check",
      "method": "GET",
      "authorizationType": "none",
      "apiKeyRequired": false,
      "requestParameters": {},
      "requestTemplates": {
+       "application/json": {
+           "password": "$input.params('password')"
+       }
      },
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {},
          "responseTemplates": {
            "application/json": ""
          }
        }
      }
    }
  ]
}
{% endcodeblock %}

Note that the _s-function.json_ file is also where you can configure if the service accepts `POST` or `PUT` or `DELETE` requests. Here we are only interested in `GET`.

You can easily tailor the `requestTemplates` in this file to extract whatever parameters you need in your lambda function.

## Retrieving the parameter value ##

Now back in _index.js_ you will find that the function's `event` parameter has a property `password` which is set to the value of the querystring parameter.

{% codeblock lang:javascript nodejscomponent/lib/index.js %}
module.exports.respond = function(event, cb) {
  
  var parameterValue = event.password; // the querystring parameter
  var response = {
    message: parameterValue
  };

  return cb(null, response);
};
{% endcodeblock %}

Redeploy.

     $ serverless dash deploy
     
Visit the URI.

    http://something.amazonaws.com/development/potd/check?password=P455w0rd

The response is:

```json
{
    message: "P455w0rd"
}
```

## Ready for implementation ##

We now have all the pieces we need. Serverless, Typescript, Mocha and AWS. In the next post I'll show how to wire up everything get it working.

