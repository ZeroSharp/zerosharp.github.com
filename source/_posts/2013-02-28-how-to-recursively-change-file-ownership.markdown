---
layout: post
title: "How to recursively change file ownership"
date: 2013-02-28 17:40
comments: true
categories: [powershell, windows]
description: A powershell script to recursively change the ownership of files in the current and subdirectories.
---
I recently ran into some file ownership trouble after cloning a bitbucket repository. 

The following script saved my bacon.

{% codeblock lang:powershell FixOwnership.ps1 %}
# This script recursively fixes the ownership on the files in the 
# current and subdirectories.

$acct1 = New-Object System.Security.Principal.NTAccount('Administrators')
$profilefolder = Get-Item .
$acl1 = $profilefolder.GetAccessControl()
$acl1.SetOwner($acct1)
dir -r . | set-acl -aclobject $acl1
pause
{% endcodeblock %}