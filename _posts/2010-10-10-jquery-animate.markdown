---
layout: post
title: jQuery Animate
description: jQuery is full of hidden goodies, one of which is animate. It does so much more that it says on the tin.
---

jQuery's `animate` does a lot more than just scaling and translating nodes in a
DOM and because of this it can be used to achieve some interesting effects.

## How it works

When you call `animate` on a jQuery object you typically pass a hash of
properties to change like so.

{% highlight javascript %}
// Here background colour on `#selector` fades to black.
$('#selector').animate({backgroundColor: '#000'});
{% endhighlight %}

The interesting thing is you don't have to use a selector to construct the
initial jQuery object. You can simply pass jQuery a hash and this will define
your initial properties.

{% highlight javascript %}
$({a: 0}).animate({a: 5},
  step: function() {
    console.log(a);
  }
);
{% endhighlight %}

The result of running the above example will be 0 to 5 output in your console
log. Duration still works as it would with a typical `animate`.

## A real-world application

A more realistic example that reflects something I did recently is below.

{% highlight javascript %}
$progress_bar = $('#progress_bar');

$({counter: 0, width: 0}).animate({counter: 50, width: 200},
  step: function() {
    $progress_bar.css({width: this.width});
    $progress_bar.innerText(Math.ceil(this.counter));
  }
);
{% endhighlight %}
