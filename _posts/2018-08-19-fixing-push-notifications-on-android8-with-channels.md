---
layout:     post
title:      Fixing push notifications on Android 8. Aka channels
date:       2018-08-19
categories: xamarin android
permalink: /:categories/:title.html
---
Upgrading your target SDK to 26+ (Android 8) can break your push notifications all of a sudden. Fortunatelly it´s very easy to fix.

Android 8 (API level 26) expects any notification to use a [Channel](https://developer.android.com/training/notify-user/channels). So we need to implement them but only when the API level is 26+. Channels should be ignored otherwise because they won´t be available, meaning that your app will throw an exception if you try to use them.

We will create a default channel to be used when any push notification arrives. You can actually create multiple channels but I bet most applications will be just fine with one.

{% highlight csharp %}
public static class NotificationHelper
{
    public const string ChannelId = "com.unique.string.onyourapp.default";

    // make sure you call this method only once
    public static void CreateDefaultChannel(Context context)
    {
        if (Build.VERSION.SdkInt < BuildVersionCodes.O) 
            return;
        
        var channel = new NotificationChannel(ChannelId, "AppName default",
            NotificationImportance.High);
        
        channel.EnableVibration(true);
        channel.LockscreenVisibility = NotificationVisibility.Public;

        var notificationManager = (NotificationManager) context
            .GetSystemService(Context.NotificationService);
        
        notificationManager.CreateNotificationChannel(channel);
    }

    // method extension to ease the creation of the nofication
    public static NotificationCompat.Builder SetChannelIdIfNeeded(
        this NotificationCompat.Builder builder,
        string channelId)
    {
        if(Build.VERSION.SdkInt >= BuildVersionCodes.O)
            builder.SetChannelId(channelId);
        
        return builder;
    }
}
{% endhighlight %}

I´ts a good idea to create the default channel at app start, either on your main activity or your custom Application `OnCreate()` override:

{% highlight csharp %}
public override void OnCreate()
{
    base.OnCreate();
    NotificationHelper.CreateDefaultChannel(this);
}
{% endhighlight %}

And last, but not least, you need to make and slight modification to the code that creates the notification.

Before:

{% highlight csharp %}
var notificationBuilder = new NotificationCompat.Builder(this)
    .SetSmallIcon (Resource.Drawable.ic_notification)
    .SetContentTitle(title)
    .SetContentText(message)
    .SetAutoCancel(true)
    .SetContentIntent(pendingIntent);
{% endhighlight %}

After:

{% highlight csharp %}
var notificationBuilder = new NotificationCompat.Builder(this)
    .SetChannelIdIfNeeded(NotificationHelper.ChannelId) // method extension
    .SetSmallIcon (Resource.Drawable.ic_notification)
    .SetContentTitle(title)
    .SetContentText(message)
    .SetAutoCancel(true)
    .SetContentIntent(pendingIntent);
{% endhighlight %}

From now on your push notifications will start working again on Android 8.  
 
