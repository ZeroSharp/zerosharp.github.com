---
layout: post
title: "NuGet 1.8 Installation Problem"
date: 2012-05-25 00:26
comments: true
categories: [visualstudio, nuget]
description: A fix for a problem when installing NuGet 1.8.
---

The NuGet team just released NuGet 1.8.  Unfortunately there is an issue when upgrading it via the NuGet package manager.  The following message appears and the installation fails.

<pre>
VSIXInstaller.SignatureMismatchException: The installed version of 'NuGet Package Manager' 
is signed, but the update version has an invalid signature. Therefore, Extension Manager 
cannot install the update. 

  at VSIXInstaller.Common.VerifyMatchingExtensionSignatures(IInstalledExtension installedExtension, 
    IInstallableExtension updateExtension) 
  at VSIXInstaller.InstallProgressPage.BeginInstallVSIX(SupportedVSSKU targetAppID)
</pre>    

As explained in the [NuGet release notes](http://docs.nuget.org/docs/release-notes/nuget-1.8), one solution is to uninstall NuGet from the VS Extension Gallery before reinstalling.  Note: If Visual Studio won't allow you to uninstall the extension (the Uninstall button is disabled), then you likely need to restart Visual Studio using "Run as Administrator."

Another alternative (courtesy of [Jeff Handley](http://jeffhandley.com/)) is to install [this Visual Studio hotfix](http://connect.microsoft.com/VisualStudio/Downloads/DownloadDetails.aspx?DownloadID=38654) and the next time you start Visual Studio, updating via the package manager will work.