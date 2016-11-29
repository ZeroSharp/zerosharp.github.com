---
layout: post
title: "Serverless Framework Part 5: Pulling in a node package"
date: 2016-02-09 11:31
comments: true
categories: [serverless, aws, lambda, typescript, mocha, chai, crypto-js]
description: An introduction to the Serverless framework. Making it easy to use Amazon Lambda to build highly scalable apps cheaply. We flesh out the password generation routine with a call to the crypto-js node package.
---
This is the final part of an ongoing series about the [Serverless framework](https://github.com/serverless/serverless).

In the previous posts, the `PasswordGenerator` always returned 'Password'. Instead each date should corresponds to a new unique password. We'll make use of the [Crypto-js](https://www.npmjs.com/package/crypto-js) node package and we'll see that the AWS lambda copes just fine.

### Installing a node package ###

Pull in the crypto-js package into the serverless component.

    $ cd nodejscomponent/
    $ npm install crypto-js --save
    crypto-js@3.1.6 node_modules/crypto-js

Now we need the typescript definitions. Watch out there are two different TypeScript typings called _cryptojs_ and _crypto-js_. The first one is more complete.

    $ typings install cryptojs --ambient --save
    ? Found cryptojs typings for DefinitelyTyped. Continue? Yes
    Installing cryptojs@~3.1.2 (DefinitelyTyped)...

    cryptojs
    └── (No dependencies)

I'm not sure why, but there's no `export` in the typings file for _cryptojs_. Add the following to the bottom of the _cryptojs.d.ts_ file. 

{% codeblock lang:javascript nodejscomponent/typings/main/ambient/cryptojs.d.ts %}
  ...
declare module "crypto-js" {
    export = CryptoJS;
}
{% endcodeblock %}  

### Improved PasswordGenerator ###

{% codeblock lang:javascript nodejscomponent/src/passwordOfTheDay.ts %}
/// <reference path="../typings/main.d.ts" />
import CryptoJS = require("crypto-js"); 

export function checkPotd(password : string) : boolean
{
    return new PasswordGenerator().check(password);
}

export class PasswordGenerator 
{
	generate(date: Date) : string 
	{
        // Get the current date as a YYYYMMDD string
        var yyyy = date.getFullYear().toString();
        var mm = (date.getMonth()+1).toString(); // getMonth() is zero-based
        var dd  = date.getDate().toString();
        var plain = `${yyyy}${mm}${dd}`;

        // Using AES CTR with 32 byte key and iv ensures the encrypted string is not too long
        // See http://stackoverflow.com/a/13298019/1077279
                
        var key = CryptoJS.enc.Hex.parse('108c786594543687891374723e809ec5e475a8361f7ad82df04e91ba2c139321');
        // Use a different initialization vector each time by using the date as part of the vector
        var iv  = CryptoJS.enc.Hex.parse(plain + '3a8fe4440be1e113a271574f379d70a76c3477aaff036d1e83fcd4b9');
        var options = { mode: CryptoJS.mode.CTR, padding: CryptoJS.pad.NoPadding, iv: iv };

        var encrypted = CryptoJS.AES.encrypt(plain, key, options);
        
 		return encrypted.ciphertext.toString();
	}	
	
	check(password : string) : boolean 
	{
        // check the value matches today's password of the day
		return password == this.generate(new Date());
	}
} 
{% endcodeblock %}

### Run the tests ###

We expect the tests to fail now since we are no longer returning the same password.

```sh
$ npm test
> @0.0.1 pretest /Users/ra/Projects/Coprocess/serverlessPotd/nodejscomponent
> tsc

> @0.0.1 test /Users/ra/Projects/Coprocess/serverlessPotd/nodejscomponent
> mocha ./lib/test

  Generator
    #generate
      1) should generate the password
    #check
      ✓ should return false when the password is incorrect
    #check
      2) should return true when the password is correct

  1 passing (16ms)
  2 failing

  1) Generator #generate should generate the password:
     Error: Expected 'Password' but was bb4bde4d76b055
      at Context.<anonymous> (lib/test/passwordOfTheDayTest.js:12:23)

  2) Generator #check should return true when the password is correct:
     Error: Expected 'true' but was false
      at Context.<anonymous> (lib/test/passwordOfTheDayTest.js:28:23)

npm ERR! Test failed.  See above for more details.
```

### Update and improve the tests ###

It's getting a little more complicated so let's pull in [chai](http://chaijs.com/) which is a pretty assertions library.

```sh
$ npm install chai --save-dev
chai@3.5.0 node_modules/chai
├── assertion-error@1.0.1
├── type-detect@1.0.0
└── deep-eql@0.1.3 (type-detect@0.1.1)
```
And the Typescript definitions for chai.


```sh
$ typings install chai --save --ambient
? Found chai typings for DefinitelyTyped. Continue? Yes
Installing chai@~3.4.0 (DefinitelyTyped)...

chai
└── (No dependencies)
```

Now let's flesh out the tests for the `PasswordGenerator` class.

{% codeblock lang:javascript nodejscomponent/src/test/passwordOfTheDayTest.ts %}
/// <reference path="../../typings/main.d.ts" />
import PasswordOfTheDay = require("../passwordOfTheDay");
import Chai = require("chai");

// Tell chai that we'll be using the "should" style assertions.
Chai.should();

describe("Generator", () => {
    var subject : PasswordOfTheDay.PasswordGenerator;
    
    beforeEach(function () {
        subject = new PasswordOfTheDay.PasswordGenerator();
    });

    describe("#generate", () => {
        it("should generate the password when the date is 24th July 2010", () => {
            var date : Date = new Date(2010, 6, 24);
            var password : string = subject.generate(date); 
                       
            password.should.equal("92ab1ff89bf9af");           
        });
    });

    describe("#generate", () => {
        it("should generate a different password when the date is 25th July 2010", () => {
            var date : Date = new Date(2010, 6, 25);
            var password : string = subject.generate(date); 
                       
            password.should.equal("26a394b21800f1");           
        });
    });  
    describe("#check", () => {
        it("should return false when the password is incorrect", () => {
            var password : string = "garbage";
            var result : boolean = subject.check(password);
            
            result.should.be.false;
        });
    });

    describe("#check", () => {
        it("should return false when the password is null", () => {
            var password : string = null;
            var result : boolean = subject.check(password);
            
            result.should.be.false;
        });
    });

    describe("#check", () => {
        it("should return true when the password is correct", () => {
            var password : string = subject.generate(new Date());
            var result : boolean = subject.check(password);
  
            result.should.be.true;
        });
    });
});
{% endcodeblock %}

Run our tests:

```sh
$ npm test
> @0.0.1 pretest /Users/ra/Projects/Coprocess/serverlessPotd/nodejscomponent
> tsc

> @0.0.1 test /Users/ra/Projects/Coprocess/serverlessPotd/nodejscomponent
> mocha ./lib/test

  Generator
    #generate
      ✓ should generate the password when the date is 24th July 2010
    #generate
      ✓ should generate a different password when the date is 25th July 2010
    #check
      ✓ should return false when the password is incorrect
    #check
      ✓ should return false when the password is null
    #check
      ✓ should return true when the password is correct


  5 passing (20ms)
```

### Deploy ###

    $ serverless dash deploy
    
Now when we visit the endpoint with the correct password for today's date which happens to be _89366e6199f3_.

    https://...amazonaws.com/dev/potd/check?password=89366e6199f3
   
```json
{
    message: true
}
```

Mission accomplished! Notice that using a node package did not require any special steps on the AWS side. In fact we have not had to login to AWS since the very beginning when we created an IAM user for the project. And yet we've managed to build and deploy a cheap and scalable cloud-based service.

The [source code](https://github.com/ZeroSharp/ServerlessPotd) is on GitHub. Note the default _.gitignore_ file skips the _admin.env_ file which contains the (sensitive) AWS keys in it so don't forget to add your own.

That wraps up my series on building a small, but real-world Serverless application. In a future post, I'd like to look at providing a secured 'generate' service to allow authorized users to get today's password. Stay tuned.