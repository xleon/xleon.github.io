---
layout:     post
title:      How to use Charles Proxy in Xamarin to capture network traffic (including SSL)
date:       2018-11-16
categories: xamarin android ssl charles-proxy
permalink: /:categories/:title.html
---

Capturing network traffic between your application and your server is a handy way of checking and debugging the data you are sending and what comes back from the server. [Charles Proxy](https://www.charlesproxy.com/) shows network calls in a sequence or as a tree structure, including headers and all kind of http information like response times and payload size. It even allows you to simulate adverse network conditions and throttling, ie: an unstable 3g connection. 

I´ve been doing this forever on web development so I tried on mobile by configuring a proxy on Android emulators according to some detailed tutorials out there.

> Result: None of the tutorials worked with my Xamarin app

Well, actually those tutorials work 😜 but Xamarin needs an additional piece of code. As I don´t want to replicate wonderful posts from other developers, I recommend you follow this specific tutorial step by step: [The Android Emulator and Charles Proxy: A Love Story](https://medium.com/@daptronic/the-android-emulator-and-charles-proxy-a-love-story-595c23484e02). It´s a bit long but hey, you only need to do it once! 

## Xamarin specific bits: writting a custom HttpClientHandler

I´ve been pulling my hair big time because the tutorial didn´t work, until I tried by manually pointing to Charles Proxy from the app code.

First of all, check your local ip and charles port.  
On Charles, click menu > Local IP addresses:

<div style="text-align:center">
    <img src="/images/charles-ip.png">
</div> 

Then go to menu > Proxy > Proxy settings... to check the default port:

<div style="text-align:center">
    <img src="/images/charles-port.png">
</div> 

Setup the proxy with a custom `HttpClientHandler`:

{% highlight csharp %}
var handler = new HttpClientHandler
{
    // local ip, charles port
    Proxy = new WebProxy("192.168.0.52", 8888) 
};
{% endhighlight %}

Then use it whenever you create an `HttpClient` in your app

{% highlight csharp %}
var client = new HttpClient(handler);
// TODO: make some requests to test charles!
{% endhighlight %}

At this point everything should work and you should be able to observe all network traffic. For the shake of your mental health, make sure you only do this on debug mode and it never gets to production. 

## Bonus

__Beware__ that your app stops working if your `HttpClientHandler` is pointing to Charles but Charles is not running. To tackle this issue I personally use an environment variable that I can set to `false` when I´m not debugging network traffic:

{% highlight csharp %}
// http proxy
if (Environment.GetEnvironmentVariable("UseHttpProxy") == "true")
{
    var handler = new HttpClientHandler
    {
        Proxy = new WebProxy(
            Environment.GetEnvironmentVariable("ProxyHost"), 
            int.Parse(Environment.GetEnvironmentVariable("ProxyPort") 
                ?? throw new InvalidOperationException()))
    };

    _client.SetHandler(handler);
}
{% endhighlight %}

To set env variables on Android, you have to create a text file in your Android project root, call it _env.txt_ (it can be any name) and set its _Build Action_ to `AndroidEnvironment`:

<div style="text-align:center">
    <img src="/images/env_txt.png">
</div> 

   


