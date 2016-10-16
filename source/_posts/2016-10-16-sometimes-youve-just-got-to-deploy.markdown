---
layout: post
title: "Sometimes you've just got to deploy"
date: 2016-10-16 18:44
comments: true
categories: [testing, nunit, c#]
description: A technique for temporarily skipping a unit test.
---

Sometimes the deadline has arrived and you still have some failing tests. After a discussion with the dev team, you decide to deploy anyway and fix the bugs for the next release. You need to get the build server to ignore the tests.

One way is just to mark the test with the `[Ignore]` attribute.

```c#
[Test]
[Ignore] // TODO: Fix this test before the next release!
public void Test()
{
    // Some failing test code...
}
```

After the weekend, everyone forgets about the ignored tests and they never get fixed.

Instead, I like to do this.
```c#
[Test]
public void Test()
{
    if (DateTime.Now < new DateTime(2016, 10, 17))
        Assert.Ignore("Temporarily ignored until October 17.");
    // Some failing test code...
}
```

This is a fairly rare occurrence for my team, so the above approach is sufficient and works with all test frameworks. But if you want to go further [Richard Slater shows how to create an NUnit attribute](https://www.amido.com/code/conditional-ignore-nunit-and-the-ability-to-conditionally-ignore-a-test/).
