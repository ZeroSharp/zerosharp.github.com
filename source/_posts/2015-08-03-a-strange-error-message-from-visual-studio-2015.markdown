---
layout: post
title: "A strange error message from Visual Studio 2015"
date: 2015-08-03 15:35
comments: true
categories: [visual studio, c#]
description: A 'procedure entry point could not be located' because of the naming of an assembly when debugging with VS 2015.
---
When I recently upgraded to Visual Studio 2015, everything seemed to go very smoothly except that whenever I debugged my main application I got a dialog window with the following strange error:

    The procedure entry point could not be located in the dynamic link library C:\...\bin\Debug\netutils.dll.

After pressing `OK` everything seemed to work as normal.

After a considerable number of dead ends, I finally worked out that changing the name of the _NetUtils.dll_ assembly fixes the problem. It seems that Visual Studio 2015 gets confused with a Windows system assembly with the same name. I don't know why it was never a problem with Visual Studio 2013, but I renamed the assembly and now everything is working fine.