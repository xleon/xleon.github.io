---
layout:     post
title:      Going back to the nineties
date:       2017-02-08
summary:    Or how to make a blog without a database
categories: jekyll
---
I´m a developer and I´ve always wanted to have my own blog. I like the idea of helping others by telling them how to make things easier. So here I am, building a new blog for the fifth time in my life.  

In the past I built my site with [Wordpress](https://wordpress.com/) (php), [Umbraco](https://umbraco.com/) (.NET), [Django](https://www.djangoproject.com/) (python) and finally with [Tumblr](https://www.tumblr.com).
All of them except Tumblr required a hosting provider, manual setup and a monthly fee. Tumblr is nice and easy, but [my contents didn´t appear on google search](https://www.distilled.net/blog/seo/seo-for-tumblr-blogs/), possibly due to my ignorance or maybe it´s just Tumblr, who knows...

All of them provide a nice content management web interface to edit your stuff. Migrating from one to another required almost starting from scratch, but that wasn´t a problem because I like adopting and learning new tools/frameworks and improving my site was the best excuse for that.

I´m really tired of managing servers, hostings accounts and payments, even ftp deployment (is anyone still doing this?), installing content management systems and spening too much of my time on it. That´s fine for client work, but I wanted something easier for myself this time. 

## I found Jekyll

[Jekyll](https://jekyllrb.com/) is a static site generator based on templates that you run on your own computer. You just need to create and edit markdown files. One file for each page or blog post. That´s all. Simple and awesome.

It works with ruby and you can install it and run it in about 1 minute (if you don´t get any errors like [this one](https://jekyllrb.com/docs/troubleshooting/#jekyll-amp-mac-os-x-1011)). When Jekyll builds your site you get a folder ("_site" by default) with static content (no database, no server scripting) that you can deploy to any cheap or free hosting. 

**The concept of static content generators is not new. So what´s the point?**

Well, turns out that the underlaying tech on [Github Pages](https://pages.github.com/) is Jekyll, and this means that:

- You can edit your site/blog in your local computer and push changes to a github repository. Github pages will automatically generate the static content for you, no any other action required.
- You can edit your site/blog online in the github site. Again, the static content will be generated for you once you save a file.
- Obviously, you get CVS for your site content 
- It´s free

Jekyll has a bunch of [community free themes](https://github.com/jekyll/jekyll/wiki/Themes) that you can easily install, normally forking its github repo and then setting up your own preferences on the `_config.yml` file.

This post is not about telling you how to use Jekyll, as there are awesome tutorials out there like [this one](https://jekyllrb.com/docs/quickstart/) or [this one](http://jmcglone.com/guides/github-pages/). [Readme files on theme](https://github.com/johnotander/pixyll) repos are also a good source of information.

Hope you find this useful.

Cheers