---
layout: post
title: How To Send In Javascript
---

In Ruby you can and probably often do send messages in a couple of ways.

{% highlight ruby %}
'receiver'.length
'receiver'.send(:length)
'receiver'.method(:length).call
{% endhighlight %}

So how do you achieve the same in Javascript?

## Sending Messages in Javascript

{% highlight javascript %}
'receiver'.length()
'receiver'['length']()
{% endhighlight %}

### Dynamic Messages

So knowing you can access methods on a receiver in Javascript using `#[]` you
can start doing some interesting things.

{% highlight javascript %}
var rainbow = {
  dark_red: '900',
  bright_red: 'f00',

  dark_green: '090',
  bright_green: '0f0',

  dark_blue: '009',
  bright_blue: '00f'
}

var color = function(name, bright) {
  var valid_color = rainbow[name + '_' + (bright ? 'bright' : 'dark')];
  if (valid_color) return '#' + valid_color();
}
{% endhighlight %}
