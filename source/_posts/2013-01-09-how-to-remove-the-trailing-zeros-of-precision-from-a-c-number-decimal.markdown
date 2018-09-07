---
layout: post
title: "How to remove the trailing zeros of precision from a C# Decimal"
date: 2013-01-09 17:45
comments: true
categories: [c#]
description: A quirky fix for C# decimal precision.
---
You may know that the C# `Decimal` type remembers any trailing zeros. So for instance:

    Console.WriteLine(123.45m);
    Console.WriteLine(123.45000m);
    Console.WriteLine(123.4500000m);

results in 

    123.45
    123.45000
    123.4500000

This is because the `Decimal` type is designed for representing a number including its accuracy.

It is tricky to find a good way of controlling this accuracy. If I have `x = 123.45000`, how can I easily remove the trailing zeros so that `x.ToString()` outputs `123.45`? 

For a while it looked like I'd have to perform some [Jon Skeet wizardry](http://stackoverflow.com/a/4298787/1077279), but there is a simpler solution. The following extension method will remove all the trailing zeros.

    public static class DecimalExtensions
    {
        public static Decimal Normalize(this Decimal value)
        {
            return value / 1.000000000000000000000000000000000m;
        }
    }

And if you need an exact number of trailing zeros, you can first `Normalize()` and then multiply.

    Console.WriteLine(123.45m.Normalize() * 1.000m); 

will output the following.

    123.45000
