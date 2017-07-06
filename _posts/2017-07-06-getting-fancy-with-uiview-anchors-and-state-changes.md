---
layout:     post
title:      Getting fancy with UIView anchors and state changes
date:       2017-07-06T23:20:00Z
summary:    
categories: xamarin ios NSLayoutConstraint autolayout
permalink: /:categories/:title.html
---

If you prefer coded user-interfaces rather than designers (Xcode Interface Builder or Xamarin designers) you will surely like this API (available from iOS9+). It´s very readable, easy to maintain and feels more natural compared to the old one.

Every `UIView` has now a number of `NSLayoutAnchor` properties
<div style="text-align:center">
    <img src="/images/anchor-properties.png" alt="anchor properties">
</div>  
<br> 
that can be used to create constraints (`NSLayoutCostraint`):
<div style="text-align:center">
    <img src="/images/constraint-methods.png" alt="constraint methods">
</div>  
<br> 

Creating a constraint does not mean it will be active by default and you´ve got do it explicitly:

{% highlight csharp %}
var c = origin.CenterXAnchor.ConstraintEqualTo(target.CenterXAnchor);
c.Active = true;

// or just
origin.CenterXAnchor
    .ConstraintEqualTo(target.CenterXAnchor)
    .Active = true;

// activate multiple constraints at once:
NSLayoutConstraint.ActivateConstraints(new[]{
    origin.CenterXAnchor.ConstraintEqualTo(target.CenterXAnchor),
    origin.CenterYAnchor.ConstraintEqualTo(target.CenterYAnchor)
});
{% endhighlight %}

It´s really easy to create `UIView` method extensions. For instance, to center a view in its parent (following the above sample):

{% highlight csharp %}
public static NSLayoutConstraint[] CenterIn(this UIView origin, 
    UIView target, bool activate = true)
{
    origin.TranslatesAutoresizingMaskIntoConstraints = false;
    var contraints = new[]
    {
        origin.CenterXAnchor.ConstraintEqualTo(target.CenterXAnchor),
        origin.CenterYAnchor.ConstraintEqualTo(target.CenterYAnchor)
    };

    if (activate)
        NSLayoutConstraint.ActivateConstraints(contraints);

    return contraints;
}

// use case
view.CenterIn(parentView);
{% endhighlight %}

Now let´s adjust a view to its parent bounds:

{% highlight csharp %}
public static NSLayoutConstraint[] FullSizeOf(this UIView origin, 
    UIView target, UIEdgeInsets edges, bool activate = true)
{
    origin.TranslatesAutoresizingMaskIntoConstraints = false;
    var contraints = new[]
    {
        origin.LeadingAnchor.ConstraintEqualTo(target.LeadingAnchor, edges.Left),
        origin.TrailingAnchor.ConstraintEqualTo(target.TrailingAnchor, -edges.Right),
        origin.TopAnchor.ConstraintEqualTo(target.TopAnchor, edges.Top),
        origin.BottomAnchor.ConstraintEqualTo(target.BottomAnchor, -edges.Bottom)
    };

    if(activate)
        NSLayoutConstraint.ActivateConstraints(contraints);

    return contraints;
}

public static NSLayoutConstraint[] FullSizeOf(this UIView origin, 
    UIView target, float margin = 0, bool activate = true) 
    => origin.FullSizeOf(target, new UIEdgeInsets(margin, margin, margin, margin), activate);

// use cases
view.FullSizeOf(parentView);
view.FullSizeOf(parentView, 10); // 10 points of margin
view.FullSizeOf(parentView, new UIEdgeInsets(5, 10, 5, 10));
{% endhighlight %}

And now let´s make a shortcut to set the width and height:

{% highlight csharp %}
public static NSLayoutConstraint[] ConstraintSize(this UIView origin, 
    float width, float height, bool activate = true)
{
    origin.TranslatesAutoresizingMaskIntoConstraints = false;
    var contraints = new[]
    {
        origin.WidthAnchor.ConstraintEqualTo(width),
        origin.HeightAnchor.ConstraintEqualTo(height)
    };

    if (activate)
        NSLayoutConstraint.ActivateConstraints(contraints);

    return contraints;
}

// use case
view.ConstraintSize(100, 100);
{% endhighlight %}

Consider playing with all anchors and `ConstraintTo...()` methods. You can do literally anything.

Those method extensions above return an array of constraints so we can deactivate/activate them later. Consider a set of constraints like a visual state we can store in a variable:

{% highlight csharp %}
var small = view.ConstraintSize(100, 80);
var big = view.ConstraintSize(200, 200, false);
{% endhighlight %}

States can get even more complex by chaining the constraints of multiple objects:

{% highlight csharp %}
var small = view1.ConstraintSize(100, 100)
    .Concat(view2.ConstraintSize(50, 50))
    .Concat(view3.ConstraintSize(30, 30))
    .ToArray();

var big = view1.ConstraintSize(200, 200, false)
    .Concat(view2.ConstraintSize(100, 100, false))
    .Concat(view3.ConstraintSize(60, 60, false))
    .ToArray();
{% endhighlight %}

## State changes

Simply deactivate the old constraints and activate the new ones:

{% highlight csharp %}
NSLayoutConstraint.DeactivateConstraints(small);
NSLayoutConstraint.ActivateConstraints(big);
{% endhighlight %}

With the basic blocks in place, let´s create a new extension to get fancy with state changes:

{% highlight csharp %}
public static void ChangeState(this UIView parentView, 
    NSLayoutConstraint[] before, 
    NSLayoutConstraint[] after, 
    double duration = 0)
{
    parentView.LayoutIfNeeded();

    NSLayoutConstraint.DeactivateConstraints(before);
    NSLayoutConstraint.ActivateConstraints(after);

    if (duration > 0)
    {
        UIView.Animate(duration, parentView.LayoutIfNeeded);
    }
}

// use cases
View.ChangeState(small, big);
View.ChangeState(big, small, duration:0.3);
{% endhighlight %}

Here´s a working [example](https://github.com/xleon/UIStackViewPlayground/blob/master/Views/AnchorPocViewController.cs):

<div style="text-align:center">
    <img src="/images/change-state.gif" alt="change state">
</div>  
<br> 



