---
layout: post
title: "Provisioning a new development machine with BoxStarter"
date: 2014-02-25 15:29
comments: true
categories: [windows, visual studio, xaf]
description: How to provision a new development machine with BoxStarter.
---
I've been playing around with [Boxstarter](http://boxstarter.org/) to configure my entire development environment with hardly any user intervention.

{% img right /images/blog/boxstarter-001.png 300 %}

Here are the steps:

1. Install Windows 8.1 on a new machine.
2. Login.
3. Open a command prompt and enter the following.
```
START http://boxstarter.org/package/nr/url?http://bit.ly/1kapDXI
```

That's it! 

Boxstarter will self-install via ClickOnce, asking for various confirmations and ultimately it will prompt you for your login password. (This gets saved and encrypted to allow for unattended reboots and re-logins during the installation). Then the real magic begins. Boxstarter downloads and installs all your tools and configures your environment, rebooting as necessary. An hour later your full development setup is installed, including Visual Studio 2013, any VS extensions, any other programs and tools, all the browsers you need, all critical Windows updates, etc. You just saved yourself a couple of days of work and a lot of hassle.

How does Boxstarter know what to install? There's a Powershell script located at that bitly address. Let's take a look at the script. 

```powershell

# Boxstarter options
$Boxstarter.RebootOk=$true # Allow reboots?
$Boxstarter.NoPassword=$false # Is this a machine with no login password?
$Boxstarter.AutoLogin=$true # Save my password securely and auto-login after a reboot

# Basic setup
Update-ExecutionPolicy Unrestricted
Set-ExplorerOptions -showHidenFilesFoldersDrives -showProtectedOSFiles -showFileExtensions
Enable-RemoteDesktop
Disable-InternetExplorerESC
Disable-UAC
Set-TaskbarSmall

if (Test-PendingReboot) { Invoke-Reboot }

# Update Windows and reboot if necessary
Install-WindowsUpdate -AcceptEula
if (Test-PendingReboot) { Invoke-Reboot }

# Install Visual Studio 2013 Professional 
cinstm VisualStudio2013Professional -InstallArguments WebTools
if (Test-PendingReboot) { Invoke-Reboot }

# Visual Studio SDK required for PoshTools extension
cinstm VS2013SDK
if (Test-PendingReboot) { Invoke-Reboot }

cinstm DotNet3.5 # Not automatically installed with VS 2013. Includes .NET 2.0. Uses Windows Features to install.
if (Test-PendingReboot) { Invoke-Reboot }

# VS extensions
Install-ChocolateyVsixPackage PowerShellTools http://visualstudiogallery.msdn.microsoft.com/c9eb3ba8-0c59-4944-9a62-6eee37294597/file/112013/6/PowerShellTools.vsix
Install-ChocolateyVsixPackage WebEssentials2013 http://visualstudiogallery.msdn.microsoft.com/56633663-6799-41d7-9df7-0f2a504ca361/file/105627/31/WebEssentials2013.vsix
Install-ChocolateyVsixPackage T4Toolbox http://visualstudiogallery.msdn.microsoft.com/791817a4-eb9a-4000-9c85-972cc60fd5aa/file/116854/1/T4Toolbox.12.vsix
Install-ChocolateyVsixPackage StopOnFirstBuildError http://visualstudiogallery.msdn.microsoft.com/91aaa139-5d3c-43a7-b39f-369196a84fa5/file/44205/3/StopOnFirstBuildError.vsix

# AWS Toolkit is now an MSI available here http://sdk-for-net.amazonwebservices.com/latest/AWSToolsAndSDKForNet.msi (no chocolatey package as of FEB 2014)
# Install-ChocolateyVsixPackage AwsToolkit http://visualstudiogallery.msdn.microsoft.com/175787af-a563-4306-957b-686b4ee9b497

#Other dev tools
cinstm fiddler4
cinstm beyondcompare
cinstm ProcExp #cinstm sysinternals
cinstm NugetPackageExplorer
cinstm windbg
cinstm Devbox-Clink
cinstm TortoiseHg
#cinstm VisualHG # Chocolatey package is corrupt as of Feb 2014 
cinstm linqpad4
cinstm TestDriven.Net
cinstm ncrunch2.vs2013

#Browsers
cinstm googlechrome
cinstm firefox

#Other essential tools
cinstm 7zip
cinstm adobereader
cinstm javaruntime

#cinst Microsoft-Hyper-V-All -source windowsFeatures
cinst IIS-WebServerRole -source windowsfeatures
cinst IIS-HttpCompressionDynamic -source windowsfeatures
cinst IIS-ManagementScriptingTools -source windowsfeatures
cinst IIS-WindowsAuthentication -source windowsfeatures

Install-ChocolateyPinnedTaskBarItem "$($Boxstarter.programFiles86)\Google\Chrome\Application\chrome.exe"
Install-ChocolateyPinnedTaskBarItem "$($Boxstarter.programFiles86)\Microsoft Visual Studio 12.0\Common7\IDE\devenv.exe"
```

Boxstarter works with [Chocolatey](http://chocolatey.org/) and you can install anything with a Chocolatey package very easily. As you can see, most of the lines begin with `cinstm` which is a shortcut for _install with Chocolatey if missing_. You will notice there are also commands for configuring Windows and IIS options. There is plenty of additional information on the [Boxstarter](http://boxstarter.org/WhyBoxstarter) documentation.

## What about DevExpress? ##

Want to install your registered CodeRush and DexExpress components? Easy. Since the installation packages are not available on chocolatey, you will have to put them on a network share accessible from the newly provisioned machine. 

Then add the following to your boxstarter script:

```powershell
# Set the following to the network location of the DevExpress installers
$pathToDevExpressComponentsSetup = "\\somewhere\DevExpressComponents-13.2.7.exe"
$pathToDevExpressComponentsSetup = "\\somewhere\DevExpressCodeRush-13.2.7.exe"

# Command line options for unattended installation
# (/EULA is not required for versions earlier than 10.2.10)
$silentArgs = "/Q /EMAIL:myaddress@company.com /CUSTOMERID:A1111 /PASSWORD:MYPASSWORD /DEBUG /EULA:accept"

# Install .NET Components
Install-ChocolateyInstallPackage "DevExpressComponents_13.2" "EXE" $silentArgs $pathToDevExpressComponentsSetup

# Install CodeRush
Install-ChocolateyInstallPackage "DevExpressCodeRush_13.2" "EXE" $silentArgs $pathToDevExpressCodeRushSetup
```

**Warning! don't put your DevExpress passwords on a public website.**

There are plenty of other ways of _launching_ Boxstarter. You can [install Boxstarter via Chocolatey](https://chocolatey.org/packages/boxstarter). You can [run Boxstarter remotely](http://boxstarter.org/InstallingPackages). If you are putting passwords in the installation script, you should choose one of the other options.

## Advantages ##

* I save time!
* I can now version control the exact development environment along with the source code!
* Onboarding new and junior developers is much quicker.
* In a pinch, I can use this method to quickly provision a development machine with Azure or Amazon Web Services

## One Last Hiccup ##

While I was testing my BoxStarter script I used the Windows 8.1 image from [http://www.modern.ie/](http://www.modern.ie/). Much later in the process I realised that Modern.IE only supplies 32-bit images and the chocolatey installer for Visual Studio extensions currently works only with 64-bit Windows.
