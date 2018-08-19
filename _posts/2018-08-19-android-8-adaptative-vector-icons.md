---
layout:     post
title:      Android 8 adaptative (vector) icons
date:       2018-08-19
categories: xamarin android
permalink: /:categories/:title.html
---

Once we change the target android version to 26 or higher, the app icon suddenly gets messed up, from a full size icon to a tiny, hard to notice thing inside a white shape (a circle on the default Google launcher). I delayed upgrading to 26+ because of this, and also beacuse I didn´t understand how the new adaptative icons work. But it´s about time, so here we go.

### How do I create it?

I´m assuming your launcher icon (app icon) is called `ic_launcher`, but it can be any name actually. Just make sure it´s the same name you set on your main launcher activity:

{% highlight csharp %}
[Activity(
        Label = "YourApp"
        , MainLauncher = true
        , Icon = "@mipmap/ic_launcher"]
public class YourInitialActivity : AppCompatActivity {}
{% endhighlight %}

Create the folder `YourAndroidProject/Resources/mipmap-anydpi-26`.  

Inside that folder create an xml file with the exact name of your launcher icon. For instance: `ic_launcher.xml`:

<div style="text-align:center">
    <img src="/images/adaptative-icon-file-structure.png">
</div> 

```
<?xml version="1.0" encoding="utf-8"?>
<!-- ic_launcher.xml content -->
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background_color" />
    <foreground android:drawable="@mipmap/foreground" />
</adaptive-icon>
```
Replace `ic_launcher_background_color` with any color you like.

Now create an SVG file with the vector tool of your choice, like Sketch, Illustrator, InkScape, etc.
The canvas should be a square of 108 x 108. However the size doesn´t really matter as we can change it later on the xml, but it´s handy to set it right for design purposes. For that I would suggest you download a template from [link](https://medium.com/google-design/designing-adaptive-icons-515af294c783) (Resources and tools). I used the Illustrator template and placed my icon inside the gray circle in the middle.

<div style="text-align:center">
    <img src="/images/adaptative-icon-template.png">
</div>  

Here you can see the same with the template grid:

<div style="text-align:center">
    <img src="/images/adaptative-icon-template-grid.png">
</div> 

### Exporting the SVG:

This will be the *foreground* graphic of your adaptative icon, so apart from the icon, everything else should be hidden, BUT YOU NEED an square shape for the icon to size correctly. The trick is to make that shape invisible, setting the alpha to 0, as per the screenshot above.

Export it to SVG, and then [convert it to an android vector xml file](http://inloop.github.io/svg2android/). Call that file `foreground.xml` and put it on the `mipmap-anydpi-26` folder. 

Now open `foreground.xml` and make sure the width and height are 108:

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">

    ...

</vector>
```

That´s it, really.

### TL;DR
Take a look at the [Official UI guideline](https://developer.android.com/guide/practices/ui_guidelines/icon_design_adaptive), a blog post about [Understanding Android Adaptive Icons](https://medium.com/google-design/understanding-android-adaptive-icons-cee8a9de93e2) and another about 
[Implementing Adaptive Icons](https://medium.com/google-developers/implementing-adaptive-icons-1e4d1795470e)

IMO, the most remarkable improvement is that the launcher icon can now be an android vector xml. That means you can take any SVG file, convert it to the android xml format with a [tool](http://inloop.github.io/svg2android/), and then use that single file in all 26+ devices.

### Can I get rid of the former `drawable/mipmap_[density]` versions?

NO. Any device using an SDK less than 26 will ignore your new adaptative icon, so you still have to provide images for the different pixel densities for those devices. 

### So what´s the point then?

1. As I said earlier, your icon will look bad on API 26+ without an adaptative icon.
2. Device OEMs can now consolidate the appearance of all icons on the launcher. This means that your icon can now [adapt to different kind of shape masks, and even be animated](https://adapticon.tooo.io/#/bg=https://i.imgur.com/iqGpQCh.png/fg=https://i.imgur.com/jceH9gr.png), looking good everywhere. One device OEM can set a circle mask on them, while another OEM can use rectangle with rounded corners.










