---
layout: post
title: "Serverless Framework - Part 2: TypeScript and Mocha"
date: 2016-01-12 08:08
comments: true
categories: [serverless, aws, typescript, mocha]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply. Here we configure the project for Typescript and Mocha.
---


Happy New Year everyone! 

This is the second part of a series about the [Serverless](https://github.com/serverless/serverless) framework. Read [the first part](/serverless-framework-part-1-up-and-running/) to get up and running.

First I'll describe the webservice I'm building. Then we'll configure our environment for Typescript and Mocha testing.

## Poor man's dual factor authentication via a password of the day ##

{% pullquote %}
I'm the technical lead for an enterprise application which is in use by about 100 large multinational corporates. As part of the installation process, we ask for a registration code which is based on the date. The customer has to call us to get the password of the day. This gives us an opportunity to engage with the customer and also gives us little more control. It's a simple form of dual factor authorization where one of the factors requires a phone call. 

In the old days, the routine for checking the validity of the password was part of the source code, but we've since moved the checking function to a web service.

It's my goal to replace this 'password of the day' check function with a Serverless module. The service will take a password as input and check that it matches the password of the day.

It's a tiny, simple, rarely-used web service but AWS lambda is still a great fit for it. Although lambda can scale if necessary, in this case {"it's about not having the hassle of administering a server."}
{% endpullquote %}

## Mocha and TypeScript ##

## New version 0.3.1 ##
{% highlight Edit: since the original version of this post, a new version 0.3.1 of Serverless was released. I have updated the tutorial below to reflect the newer version. Also, [TSD has been deprecated](https://github.com/DefinitelyTyped/tsd/issues/269) in favour of [Typings](https://www.npmjs.com/package/typings) so I've updated to use Typings instead. %}

Let's do things properly and set up a testing framework.

Make sure you're in the component folder.
    
    $ cd nodejscomponent
    
Then we'll install Mocha.    

    $ npm install mocha --save-dev
    mocha@2.4.5 node_modules/mocha
    ├── escape-string-regexp@1.0.2
    ├── commander@2.3.0
    ├── diff@1.4.0
    ├── supports-color@1.2.0
    ├── growl@1.8.1
    ├── debug@2.2.0 (ms@0.7.1)
    ├── mkdirp@0.5.1 (minimist@0.0.8)
    ├── jade@0.26.3 (commander@0.6.1, mkdirp@0.3.0)
    └── glob@3.2.3 (inherits@2.0.1, graceful-fs@2.0.3, minimatch@0.2.14)

Next we'll install TypeScript. Of course you can use plain javascript if you prefer. My background is C#: I make fewer dumb mistakes with TypeScript. It looks like there [is a typescript plugin in the pipeline](https://github.com/serverless/serverless/issues/371) which will make typescript integration even easier in the future, but for now:

    $ npm install typescript --save
    typescript@1.7.5 node_modules/typescript
    
    $ npm install typings -g
    /usr/local/bin/typings -> /usr/local/lib/node_modules/typings/dist/bin/typings.js
    typings@0.6.6 /usr/local/lib/node_modules/typings
    ├── array-uniq@1.0.2
    ├── elegant-spinner@1.0.1
    ├── thenify@3.2.0
    ├── popsicle-status@1.0.1
    ... etc ...

Initialise typings.

    $ typings init
    -> written typings.json
    
Now we add the type definitions for Mocha.

    $ typings install mocha --ambient --save
    ? Found mocha typings for DefinitelyTyped. Continue? Yes
    Installing mocha@~2.2.5 (DefinitelyTyped)...

    mocha
    └── (No dependencies)
        
We need a config file for the TypeScript compiler. This goes in the same module directory (_back/modules/potd_ in my case).

{% codeblock lang:json tsconfig.json %}
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "noImplicitAny": true,
        "sourceMap": false,
        "declaration": false,
        "outDir": "lib"
    },
    "exclude": [
        "node_modules",
        "typings/browser",
        "typings/browser.d.ts"    
    ]
}
{% endcodeblock %}

Now make a subdirectory for our TypeScript source files.

    $ mkdir src
    $ mkdir src/test
    
Next up we need to modify the package.json file to add a scripts section. Only the `"scripts"` section needs changing.

{% codeblock lang:json package.json %}
{
  "name": "potd",
  "version": "0.0.1",
  "description": "Dependencies for a Password of the day Serverless Module",
  "author": "Robert Anderson",
  "license": "MIT",
  "private": true,
  "repository": {
    "type": "git",
    "url": "git://github.com/"
  },
  "keywords": [],
  "devDependencies": {
    "mocha": "^2.3.4"
  },
  "dependencies": {
    "serverless-helpers-js": "~0.0.3",
    "typescript": "^1.7.5"
  },
  "scripts": {
    "prepublish": "tsc",
    "pretest": "tsc",
    "test": "mocha ./lib/test"
  }
}
{% endcodeblock %}

We've finished setting up everything for TypeScript and Mocha. Whenever you run `tsc`, any TypeScript files in _/src_ will get compiled to javascript in _/lib_. And running `npm test` will compile and then run any Mocha tests in _/lib/test_. 

The [source code so far](https://github.com/ZeroSharp/ServerlessPotd) is on GitHub. Note the default _.gitignore_ file skips the _admin.env_ file which contains the (sensitive) AWS keys in it so don't forget to add your own.

In the next post we'll create a TypeScript class for the guts of the lambda function which checks the password of the day, along with some corresponding Mocha tests, also written in TypeScript.
