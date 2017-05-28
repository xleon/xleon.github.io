---
layout:     post
title:      A simple page-indicator for your android view-pager
summary:    It only takes a little xml and five minutes of your time
categories: xamarin android
permalink: /:categories/:title.html
---
Every app I did needed some kind of slider, like on-boards or walkthroughs, product sliders, help tutorials, photo slide-shows, etc.

In the past I´ve done some view pagers with those circle page indicators in Android that involved using either 3rd party libraries or a ton of code. That was before Lollipop came out. This is now very obvious and ridiculously simple but I´m sharing it here in case you don´t know about it.

With a little xml we are going to convert this:

<div style="text-align:center">
    <img src="/images/tabs.png" alt="android tabs">
</div>  
<br> 

into something like this:

<div style="text-align:center">
    <img src="/images/page-indicator.png" alt="android page indicator">
</div>  
<br> 

Making the connection between the view-pager and our page indicator (`TabLayout`) takes a single line: 

{% highlight csharp %}
var viewPager = FindViewById<ViewPager>(Resource.Id.viewpager);
viewPager.Adapter = new MyPagerAdapter(SupportFragmentManager, funkyList);

var dots = FindViewById<TabLayout>(Resource.Id.dots);
dots.SetupWithViewPager(viewPager, true); // <- magic here
{% endhighlight %}

So we now need to make a `TabLayout` look like a page indicator.  
Good news are that it´s pretty easy:

{% highlight xml %}
<LinearLayout
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingBottom="@dimen/activity_horizontal_margin">
    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="250dp"
        android:layout_marginBottom="5dp"/>
    <android.support.design.widget.TabLayout
        android:id="@+id/dots"
        android:layout_width="match_parent"
        android:layout_height="26dp"
        local:tabBackground="@drawable/dot_selector"
        local:tabGravity="center"
        local:tabIndicatorHeight="0dp"
        local:tabPaddingStart="7dp"
        local:tabPaddingEnd="7dp"/>
</LinearLayout>
{% endhighlight %}

`tabPaddingStart` and `tabPaddingEnd` will define the separation between the dots.  

I´ve used a `LinearLayout` here for brevety but you could use whatever layout you need. It really doesn´t matter.

To have `TabLayout` available in your project you must install the nuget `Xamarin.Android.Support.Design`.

Now take a look at the `dot_selector.xml`. It defines the the selected/unselected states:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
  <item android:drawable="@drawable/selected_dot" android:state_selected="true"/>
  <item android:drawable="@drawable/default_dot"/>
</selector>
{% endhighlight %}

The selected state (`selected_dot.xml`):

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
  <item>
    <shape
      android:innerRadius="0dp"
      android:shape="ring"
      android:thickness="4dp"
      android:useLevel="false">
      <solid android:color="@color/page_indicator_selected_color"/>
    </shape>
  </item>
</layer-list>
{% endhighlight %}

And the default state (`default_dot.xml`), that is pretty much the same with a different color:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
  <item>
    <shape
      android:innerRadius="0dp"
      android:shape="ring"
      android:thickness="4dp"
      android:useLevel="false">
      <solid android:color="@color/page_indicator_default_color"/>
    </shape>
  </item>
</layer-list>
{% endhighlight %}

As you may guess, `android:thickness` is the size of the dot. 

That´s it!