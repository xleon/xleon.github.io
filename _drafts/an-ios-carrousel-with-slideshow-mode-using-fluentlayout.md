---
layout:     post
title:      An iOS carrousel with slide-show mode using FluentLayout
summary:    
categories: xamarin ios
permalink: /:categories/:title.html
---

I´m a big fun of doing auto-layout with code. It´s faster to read the code and understand what the constraints are doing than opening Interface-Builder and selecting constraints one by one to check out their values. 

I don´t do this in every screen, but in those cases where the UI has one or two elements, an additional .xib plus its code-behind looks overkill to me. 

With the help of the [FluentLayout](https://github.com/FluentLayout/Cirrious.FluentLayout) library I´m going to write a custom and reusable `UIScrollView` that will serve as a carrousel (or slider, view-pager, walkthrough, you name it) and then I´ll add a timer to create an automatic slide-show. 

