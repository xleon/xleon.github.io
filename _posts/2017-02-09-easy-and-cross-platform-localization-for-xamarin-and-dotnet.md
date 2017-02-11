---
layout:     post
title:      Easy and cross-platform localization (Xamarin & .NET)
date:       2017-02-09
summary:    Share locales from a PCL. Get up and running in no time
categories: xamarin dotnet localization i18n
---

Sooner or later we all need to localize an app in multiple languages. Each platform provides its own way of localizing strings and you may wonder how to share locale files between them. 

If you are a .NET dev, [Resx](https://msdn.microsoft.com/en-us/library/ekyft91f(v=vs.80).aspx) is probably the first thing crossing your mind, and that´s totally fine for Visual Studio. It can be implemented on [Xamarin](https://developer.xamarin.com/guides/xamarin-forms/advanced/localization/) as well but you´ll find yourself (and potentially your client) editing xml files with the following format:

{% highlight xml %}
<data name="Key" xml:space="preserve">
    <value>KeyValue</value>
    <comment>Some comments</comment>
</data>
{% endhighlight %}

At least at the time of this writting Xamarin does not provide a Resx editor. Besides, you need to write code for manually loading the correct language, based normally on the current system culture. Xamarin has a [nice tutorial](https://developer.xamarin.com/guides/xamarin-forms/advanced/localization/) on how to do it. You´ll find even iOS, Android and win phone concrete implementations and a bunch of useful details. They also explain how to automatically bind translations in XAML creating a `IMarkupExtension` class. 

The problem is:

- I don´t want to repeat all that process in every project, or even copy-paste across projects and platforms.
- I just want a library to do it for me
- I don´t like/want Resx files and I don´t want to have a client editing them


## Hello I18N-Portable

I love simplicity and that´s why I created a lightweight utility for this matter. My idea was based on two concepts:

- Simpler locale files (similar to java .properties format)
- Straightforward setup and initialization

### Install

I18N-Portable is a [nuget package](https://www.nuget.org/packages/I18NPortable/)

### Locales

A locale is just a [locale].txt file with `key=value` pairs. You place these files on a PCL project once, under a directory called "Locales". They will be shared across platforms:

    # this is a comment
    key1 = one
    key2 = two
    key3 = translation with \nmultiple \nlines


### Setup

{% highlight csharp %}
I18N.Current
    .SetFallbackLocale("en")
    .Init(GetType().GetTypeInfo().Assembly);
{% endhighlight %}

That´s all you need. Simple enough?  
 
_You put this line at your PCL code, normally in any method executed during app initialization._

What did just happen?

The `Init()` method tells I18N-Portable which assembly is hosting the locales. Now the library figures out the current system culture and tries to load a matching locale. If you didn´t provide a locale for the current culture, it will fallback to english ("en.txt"). 

From now on you can get any translation with a string method extension from anywhere in your app PCL and/or platform projects:

{% highlight csharp %}
var translation = "key1".Translate(); // one
{% endhighlight %}

### Can I use this with my awesome Mvvm framework?

Sure. You can bind any text view to a particular key, from XAML (Xamarin.Forms, UWP, etc) and classic Android/iOS if your Mvvm framework implements bindings:

XAML:
{% highlight xml %}
<Button Content="{Binding [key]}" />
{% endhighlight %}

Xamarin.Forms XAML:
{% highlight xml %}
<Button Text="{Binding [key]}" />
{% endhighlight %} 

Android with MvvmCross:
{% highlight xml %}
<TextView local:MvxBind="Text [key]" />
{% endhighlight %} 

iOS with MvvmCross:
{% highlight csharp %}
bindingSet.Bind(view).To("[key]");
{% endhighlight %} 

To make this work, a simple indexer is needed in your ViewModel (I usually do this in a BaseViewModel once):

{% highlight csharp %}
public string this[string key] => key.Translate();
{% endhighlight %} 

More handy stuff and details are available at the [github repo](https://github.com/xleon/I18N-Portable)

Cheers!