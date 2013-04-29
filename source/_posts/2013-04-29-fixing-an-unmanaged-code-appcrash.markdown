---
layout: post
title: "Fixing an unmanaged code AppCrash"
date: 2013-04-29 16:21
comments: true
categories: c# apollodb bug
description: Debugging an AppCrash related to incorrect string marshalling in an external dll call.
---
This post is the result of a recent bug hunt in which I came across a tricky bug, found a debugging switch I'd completely forgotten existed and learned a little about calling `extern` string functions from C#.

I love bug hunting. It's like a murder mystery: you've got your suspects and you try to eliminate them one at a time until, as a famous bug hunter said: 

{% blockquote Sherlock Holmes, The Sign of Four %}
... when you have eliminated the impossible, whatever remains, however improbable, must be the truth.
{% endblockquote %}

Between about 1995 and 2006, I used a data library called Apollo almost every day. It was a bunch of C++ drivers for dBase files with some more advanced options for encryption and indexing and was a popular option for [Clipper](http://en.wikipedia.org/wiki/Clipper_(programming_language)) programmers. I joined a software project which was based on Clipper and Apollo in 1995. Apollo went through many different incarnations SuccessWare, Luxent, Vista, ApolloDB. All of these companies were essentially providing wrappers for different languages (Delphi, .NET) but the core C++ drivers always remained more or less the same [and it's still going](http://www.apollodb.com/products.asp), but it's not really actively maintained.

Fast forward to 2013 and we have a legacy console utility for migrating data from the old format, which traverses the Apollo tables and converts the data to our model (DevExpress XPO objects). This code hardly ever changes, but when I converted all our core libraries to .NET 4.5., I found I had to jump in and fix it one last time.

When I tried to run the upgraded .NET 4.5 library I got a mysterious app crash.

{% img /images/blog/appcrash/app-crash-001.png %}

The application would then close without any error message or stack trace. Nothing I did was allowing me catch any exception. None of the signatures, e.g., the `c0000374` exception code show up on Google. It shows up in the Windows event log, but apparently EventID 1000 is a very generic error message.

{% img /images/blog/appcrash/app-crash-002.png %}

I had a suspicion the problem was something to do with the Apollo assembly and I also knew that Apollo was not all managed code. I stumbled over the _Enable native code debugging_ in the project settings. I've never used this setting. (My next approach would have been to use tracing to try to pinpoint the location of the crash.)

{% img /images/blog/appcrash/app-crash-003.png %}

Visual Studio really impresses with its debugging capabilities. When we run again, we get a stack trace and an error message.

{% img /images/blog/appcrash/app-crash-004.png %}

Well Google didn't seem to have much to say about _'This may be due to a corruption of the heap'_. But the problem seems to do with `apolloTable.FieldName(i)`. With a decompiler I had a look at its definition.

{% codeblock lang:csharp %}
public string FieldName(short fieldNum)
{
	//...
    return ApolloAPI.sx_FieldName(fieldNum);
    //...
}
{% endcodeblock %}

Let's find `ApolloAPI.sx_FieldName(fieldNum);`

{% codeblock lang:csharp %}
[DllImport("SDE7.dll", CharSet=CharSet.Ansi, ExactSpelling=true)]
public static extern string sx_FieldName(short uiFieldNum);
{% endcodeblock %}

Now, knowing that SDE7.dll is written in C++, I guessed it might be having trouble with the return value being a `string`. The C++ memory management of the returned string might be getting in the way. [A bit of StackOverflow](http://stackoverflow.com/a/8242828/1077279) gave me this trick: declare the return type as `IntPtr` and use `Marshal.PtrToStringAnsi()` to get a C# `string` from the pointer. (It seems that .NET 4.5 is stricter about marshalling than earlier frameworks. Perhaps someone can enlighten me why the error did not occur with .NET 4.0?) I wrote a new extension method for `ApolloTable` and changed the code to use `apolloTable.SafeFieldName(i)` instead.

{% codeblock lang:csharp %}
    public static class TApolloTableExtensions
    {
        [DllImport("SDE7.dll", CharSet = CharSet.Ansi, ExactSpelling = true)]
        public static extern IntPtr sx_FieldName(short uiFieldNum);

        public static string SafeFieldName(this ApolloTable apolloTable, short fieldNum)
        {
        	//...
            IntPtr strPtr = sx_FieldName(fieldNum);
            return Marshal.PtrToStringAnsi(strPtr);
            //...
        }
    }
{% endcodeblock %}

And bingo! The application now runs without error.





