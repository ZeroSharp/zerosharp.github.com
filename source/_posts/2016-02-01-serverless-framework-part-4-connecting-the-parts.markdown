---
layout: post
title: "Serverless Framework - Part 4: Connecting the parts"
date: 2016-02-01 09:44
comments: true
categories: [serverless, aws, typescript, mocha]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply. We connect up all the parts, Serverless, Typescript, Mocha and AWS.
---
This is part of an ongoing series about the [Serverless framework](https://github.com/serverless/serverless): [Part 1](/serverless-framework-part-1-up-and-running/), [part 2](/serverless-framework-part-2-typescript-and-mocha/), [part 3](/serverless-framework-part-3-the-guts/). 

## New version 0.3.1 ##
{% highlight Edit: since the original version of this post, a new version 0.3.1 of Serverless was released. I have updated the tutorial below to reflect the newer version. Also, [TSD has been deprecated](https://github.com/DefinitelyTyped/tsd/issues/269) in favour of [Typings](https://www.npmjs.com/package/typings) so I've updated to use Typings instead. %}
All parts have been updated for the latest version of the framework 0.3.1.

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
/// <reference path="../../typings/main.d.ts" />
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

And lets visit that URI 

    https://rhnjv4ms2b.execute-api.eu-west-1.amazonaws.com/development/potd/check?password=nonsense

```json    
{
    message: false
}
```
   
    https://rhnjv4ms2b.execute-api.eu-west-1.amazonaws.com/development/potd/check?password=Password

```json    
{
    message: true
}
```

Rock and roll. A working password checker running on Lambda in the Amazon cloud.

Next up - we'll extend the `PasswordGenerator` class to pull in a node package and generate a better password.

The [source code so far](https://github.com/ZeroSharp/ServerlessPotd) is on GitHub. Note the default _.gitignore_ file skips the _admin.env_ file which contains the (sensitive) AWS keys in it so don't forget to add your own.
