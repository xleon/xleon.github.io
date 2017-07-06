---
layout:     post
title:      Some UIStackView magic
summary:    
categories: xamarin ios NSLayoutConstraint autolayout
permalink: /:categories/:title.html
---

The glory of the `UIStackView` is not only the _stack_ part, as stacking elements is easy to do with auto-layout, but the fact that it will udpate all constraints for you when an element is shown/hidden or added/removed. That brings a new way of handling states in a single view controller and also helps on creating custom components. I will explore some real use cases here.