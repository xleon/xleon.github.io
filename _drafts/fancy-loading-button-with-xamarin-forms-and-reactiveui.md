---
layout:     post
title:      Fancy loading button with Xamarin.Forms and ReactiveUI 
categories: xamarin.forms reactiveui
permalink: /:categories/:title.html
---

I needed a button that reacts to a loading state (meaning an `ICommand` executing), showing the corresponding feedback by changing its color and addding an activity indicator on top of it. This can be achieved by creating some RxUI bindings on the view code-behind, but given this kind of button will be reused extensively all over the app, I came up with a custom control that handles its own loading and enabled states, without losing the ability to be customized with XAML styles. 