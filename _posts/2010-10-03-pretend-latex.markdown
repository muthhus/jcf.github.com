---
layout: post
title: Pretend Textile
---

I've been working on a new blogging system recently with the intention of
powering [jamesconroyfinn.com](http://jamesconroyfinn.com) using it. It
featured a unique design per page and a giant footer with a 'slick' jQuery
scrollable archive. It was a real example of a modern, swanky blog that
impresses every visitor lucky enough to browse it's shiny pages.

While working on this ultra-modern blogging engine I had to write a letter and
used [LaTeX](http://www.latex-project.org/) to do so. LaTex is a great way to
produce clean, professionally typeset documents and in my opinion the results
look fantastic.

Then I thought in more depth about what I wanted from this blogging system I was
writing. What did I want to acheive by reinventing the wheel?

* Simple navigation
* A distinctive look that would be recognisable almost immediately
* Something that shows what I'm about
* A way to share my thoughts and discoveries

And this thinking led me to realise that using jQuery and Rails was complete
overkill and was only distracting me from achieving what I really wanted to,
what all of this was about. Writing about things I'm passionate about and
sharing what I know about tools and technology I've used in the past.

So here we are. Pretending Textile...

I'm using [jekyll](http://github.com/mojombo/jekyll),
[markdown](http://daringfireball.net/projects/markdown/) in my posts,
[pygments](http://pygments.org/) for syntax highlighting, and
[Google Fonts](http://code.google.com/webfonts) among other things to make my
site look like it was build using LaTex.

## What it looks like

Code blocks:

{% highlight ruby %}
class Tex
  # always looks clean
  def self.clean?
    true
  end
end
{% endhighlight %}

### Lorem Ipsum

Lorem ipsum dolor sit amet, consectetur magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur.  Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

Blockquotes:

> Someone who said something
