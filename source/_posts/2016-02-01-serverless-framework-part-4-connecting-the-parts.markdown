---
layout: post
title: "Serverless Framework - Part 4: Connecting the parts"
date: 2016-02-01 09:44
comments: true
categories: [serverless, aws, typescript, mocha]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply. We connect up all the parts, Serverless, Typescript, Mocha and AWS.
---
This is part of an ongoing series about the [Serverless framework](https://github.com/serverless/serverless): [Part 1](/serverless-framework-part-1-up-and-running/), [part 2](/serverless-framework-part-2-typescript-and-mocha/), [part 3](/serverless-framework-part-3-the-guts/).

## The Password of the Day Generator class ##

First up we need a class to generate and check the password of the day. For the moment, let's pretend the password of the day is always the string _"Password"_. Put the following typescript class in _nodejscomponent/src_.

{% codeblock lang:javascript nodejscomponent/src/passwordOfTheDay.ts %}
export function checkPotd(password : string) : boolean
{
    return new PasswordGenerator().check(password);
}

export class PasswordGenerator 
{
	generate(date: Date) : string 
	{
        // generate today's password
 		return "Password"; 
	}	
	
	check(password : string) : boolean 
	{
        // check the value matches today's password of the day
		return password == this.generate(new Date());
	}
} 
{% endcodeblock %}

Now add a mocha test for it.

{% codeblock lang:javascript nodejscomponent/src/test/passwordOfTheDayTest.ts %}
/// <reference path="../../typings/mocha/mocha.d.ts" />
import PasswordOfTheDay = require("../passwordOfTheDay");

describe("Generator", () => {
    var subject : PasswordOfTheDay.PasswordGenerator;

    beforeEach(function () {
        subject = new PasswordOfTheDay.PasswordGenerator();
    });

    describe("#generate", () => {
        it("should generate the password", () => {
            var result : string = subject.generate(new Date(2010, 6, 24));
            if (result !== "Password") {
                throw new Error("Expected 'Password' but was " + result);
            }
        });
    });
  
    describe("#check", () => {
        it("should return false when the password is incorrect", () => {
            var result : boolean = subject.check("garbage");
            if (result !== false) {
                throw new Error("Expected 'false' but was " + result);
            }
        });
    });

    describe("#check", () => {
        it("should return true when the password is correct", () => {
            var result : boolean = subject.check("Password");
            if (result !== true) {
                throw new Error("Expected 'true' but was " + result);
            }
        });
    });

});
{% endcodeblock %}

Now compile everything.

    $ cd nodejscomponent
    $ tsc
    
You will now find that there is a corresponding javascript file in the _lib_ folder

{% codeblock lang:javascript nodejscomponent/src/passwordOfTheDay.js %}
function checkPotd(password) {
    return new PasswordGenerator().check(password);
}
exports.checkPotd = checkPotd;
var PasswordGenerator = (function () {
    function PasswordGenerator() {
    }
    PasswordGenerator.prototype.generate = function (date) {
        // generate today's password
        return "Password";
    };
    PasswordGenerator.prototype.check = function (password) {
        // check the value matches today's password of the day
        return password == this.generate(new Date());
    };
    return PasswordGenerator;
})();
exports.PasswordGenerator = PasswordGenerator;
{% endcodeblock %}

And likewise for the mocha test in _lib/test_. Now to run those tests:

    $ npm test
            
    > @0.0.1 pretest /Users/ra/Projects/Coprocess/serverlessPotd/nodejscomponent
    > tsc

    > @0.0.1 test /Users/ra/Projects/Coprocess/serverlessPotd/nodejscomponent
    > mocha ./lib/test

    Generator
        #generate
        ✓ should generate the password
        #check
        ✓ should return false when the password is incorrect
        #check
        ✓ should return true when the password is correct

    3 passing (10ms)

Nice. Next, modify the main entry point of the component _index.js_.

{% codeblock lang:javascript nodejscomponent/lib/index.js %}
// Dependencies
var PasswordOfTheDay = require('./passwordOfTheDay');

module.exports.respond = function(event, cb) {

  var result = PasswordOfTheDay.checkPotd(event.password);  
  var response = {
    message: result
  };

  return cb(null, response);
};
{% endcodeblock %}

Notice how we make use of `event.password` which is the parameter we configured in [part 3](/serverless-framework-part-3-the-guts/) in the `s_function.json` file.

Let's deploy!

    $ serverless dash deploy
    _______                             __
    |   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
    |   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
    |____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
    |   |   |             The Serverless Application Framework
    |       |                           serverless.com, v0.1.5
    `-------'

    Serverless: Select the assets you wish to deploy:
        nodejscomponent - potd - check
        function - nodejscomponent/potd/check
        endpoint - nodejscomponent/potd/check@potd/check~GET
        - - - - -
    > Deploy

    Serverless: Deploying functions in "development" to the following regions: eu-west-1  
    Serverless: ------------------------  
    Serverless: Successfully deployed functions in "development" to the following regions:   
    Serverless: eu-west-1 ------------------------  
    Serverless:   nodejscomponent/potd/check: arn:aws:lambda:eu-west-1:962613113552:function:serverlessPotd-nodejscomponent-potd-check:development  

    Serverless: Deploying endpoints in "development" to the following regions: eu-west-1  
    Serverless: Successfully deployed endpoints in "development" to the following regions:  
    Serverless: eu-west-1 ------------------------  
    Serverless:   GET - potd/check - https://rhnjv4ms2b.execute-api.eu-west-1.amazonaws.com/development/potd/check  

And lets visit that URI 

    https://[...]amazonaws.com/development/potd/check?password=nonsense

```json    
{
    message: false
}
```

    https://[...]amazonaws.com/development/potd/check?password=Password

```json    
{
    message: true
}
```

Rock and roll. A working password checker running on Lambda in the Amazon cloud.

Next up - we'll extend the `PasswordGenerator` class to pull in a node package and generate a better password.