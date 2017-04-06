---
layout:     post
title:      UWP mobile side loading
date:       2017-04-06
summary:    The key details
categories: uwp mobile side-loading
---
Installing universal Windows apps outside of the store is a bit different than iOS or Android. [HockeyApp documentation](https://support.hockeyapp.net/kb/client-integration-windows-and-windows-phone/how-to-sideload-uwp-applications) explains how to do it but they are missing some important (and perhaps frustrating) details that I will explain below.

Actually you don´t need HockeyApp to side-load Windows 10 apps as you can upload your build wherever you want, but obviously with HockeyApp you get many more
features for free, like crash reporting, distribution management, event logging, integration with Visual Studio, etc. [Visual Studio Mobile Center](https://mobile.azure.com) is the next generation of HockeyApp but as of today it doesn´t support Windows Universal apps.

## Compile a release build for mobile

Not much to add to the HockeyApp doc in this step. Check it out [here](https://support.hockeyapp.net/kb/client-integration-windows-and-windows-phone/how-to-sideload-uwp-applications#build-application), 
but bear in mind that in the last step (architecture check-boxes), you should only check ARM if your app is not targeting desktop devices. The bundle size will decrease a lot.

## Upload the build to HockeyApp 

The result of the previous step is a new folder (i.e: _YourApp_1.0.0.0_Test_) in the _AppPackages_ directory of your UWP project with the following contents:  

<div style="text-align:center">
    <img src="/images/uwp-release.png" alt="uwp release files">
</div>  

You´ve got two options here:

1. Upload the single .appxbundle file
2. Zip the whole directory and upload it to HockeyApp (this is the same thing Visual Studio does for you when right clicking on the project > HockeyApp > Distribute app...)

Whether you choose 1 or 2 will affect how users install the app:  

If you choose 1), installing the app is easier, as the user will just click the "Download" button in the HockeyApp web page, and then click the downloaded file to install it. 

<div style="text-align:center">
    <img src="/images/hockeyapp-download.png" alt="hockeyapp download">
</div>  
<br>

Going for the option 2 means that a user will download a zip to the mobile device, opening that zip and then tapping on the .appxbundle file to finally install the app. Windows 10 Mobile cannot open zips out of the box, and so a third party app like [Zip Archiver](https://www.microsoft.com/es-es/store/p/zip-archiver/9wzdncrd1l2f) should be installed first.

Why on earth would you want to make things harder with a zip file and why is it the default option in Visual Studio? See below on "Installing dependencies"

## Upload symbols to HockeyApp

To see better crash information and stack traces, right after the previous step, drag the _.appxsym_ file to the HockeyApp page. There is also a button to "Upload build or sym":

<div style="text-align:center">
    <img src="/images/symbols.png" alt="upload symbols to hockeyapp">
</div>  
<br>

## Device setup

To install any app compiled with a test certificate or coming from an untrusted source (like HockeyApp), the user needs to enable Windows developer mode on the device. Go to settings > Update & Security > For developers and select "Developer mode":

<div style="text-align:center">
    <img src="/images/dev-mode.png" alt="windows developer mode">
</div>  
<br>

## Installing dependencies

When you debug your app through a usb connected device, Visual Studio will install all dependencies for you in the background and everything will just work. However, other users trying to install your app will open the .appxbundle, install the app and it won´t appear anywhere in the device. Windows won´t let them know something wrong happened (the installation failed silently without errors). 

This happens when dependencies are not installed on the user´s device because they don´t get installed automatically (BTW, a simple "please, install dependencies" alert would have saved me many wasted hours thinking that I did something wrong with the app bundle package). 

Dependencies need to be installed usually once per device, unless you release a new version of your app that targets a newer version of Windows. In that case users will need to install them again. It´s a good idea to inform the users and testers about these details before they try to install for the first time. 

Installing dependencies involes having them on the device and then clicking/tapping on each one to confirm installation. 
They are located in the "Dependencies" folder inside the zip you created previously:

<div style="text-align:center">
    <img src="/images/uwp-depencencies.png" alt="uwp dependencies">
</div>  

_Keep in mind that if your release build is composed by the single .appxbundle file, you are assuming the user already installed these dependencies._

## Installing the application

Last but not least, tap on the .appxbundle to install the app. Everything should work now. For the next version release, this is the only step needed.