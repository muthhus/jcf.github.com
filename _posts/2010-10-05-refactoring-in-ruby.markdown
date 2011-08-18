---
layout: post
title: Refactoring in Ruby
description: "Writing code for machines is a craft. Writing code for humans is an art and Refactoring: Ruby Edition is a fantastic resource that I found to be quite inspirational."
---

After reading [Refactoring: Ruby Edition][] by [Jay Fields][] a colleague
reminded of a very old open Q&A session at the end of a [Snakes and Rubies][]
event with the creator of Rails, [David Heinemeier Hansson][], and the
co-creator of the Django framework, [Adrian Holovaty][].

After some debate it was agreed that a significant difference between Ruby and
Python is that in Python there is one way to do something, in Ruby there are
many. David made a valid point however when he argues that of all the ways of
doing something in Ruby there is generally one that feels right, an
implementation that is more beautiful and that's where [Refactoring: Ruby
Edition][] proves to be an excellent read.

Refactoring code isn't something most programmers enjoy but it is something I
like to do when the time is right.

## When to Refactor

It's not practical or pragmatic spending all of your time refactoring code, but
there are occasions where it makes sense to remove some code smell and make the
process of changing or extending business logic easier.

Jay Fields quotes a guideline [Don Roberts][] gave him. "The Rule of Three"
states the first time you do something you just do it. The second time you do
something similar you wince at the duplication. The third time you refactor.

### Refactor When Adding New Functionality

Whether it's to simply understand something you or someone else has written in
the past it can be helpful to understand some complicated code and refactor it
to make it simpler and more easily understood.

### Refactor When You Need to Fix a Bug

When it's time to fix something that doesn't work it can be a good time to
consider simplification through refactoring, although not always!

### Refactoring During Code Reviews

I'm a big fan of code reviews and genuinely appreciate constructive criticism
from colleagues. Sharing knowledge and experience only improves a team and is a
logical time to consider refactoring.

## Time to Refactor

Consider a babe. There might be many babes but all babes have some attributes
your boss is interested in. As a programmer our boss wants us to work out how
babes work and makes sure our application can predict their behaviour in order
to make lots and lots of money.

So someone creates a babe based solely upon what they know about babes. It turns
out they don't know that much but everyone in the team pools knowledge and
eventually after several interactions we know enough about babes to keep the
board of directors happy.

<script src="https://gist.github.com/965202.js"></script>

Now we're doing a code review and looking at how a babe works. There's some
repetition in the `#dance` and `#speak` methods and the `#in_my_league?` method
has grown in to something quite difficult to understand.

We're working with Ruby, which means we can do almost anything we want to a babe
but let's remember what DHH said, there's always a beautiful way to do something
and that's what we want. Beautiful babes.

## Initialisation

Let's start with the `#initialize` method. It takes a whole lot of arguments, in
a very specific order, which makes it quite difficult to work with.

<script src="https://gist.github.com/965216.js"></script>

Using a hash of optional attributes gives us named arguments. Much nicer when
creating a new babe but not ideal when you don't aren't sure which attributes
in a babe matter to the boss!

So we could define valid attributes and only extract those using `#slice`.

<script src="https://gist.github.com/965218.js"></script>

Better but not beautiful, yet. Named arguments are great but not validating them
doesn't feel right in my opinion.

A number of methods in Rails use `#assert_valid_keys` to ensure only valid
options are passed to a method and we can do the same.

<script src="https://gist.github.com/965227.js"> </script>

### Bad Metamonkey!

Refactoring does not mean "show me your Ruby foo!" Temptation to add crazy
methods like I do below is often best resisted.

<script src="https://gist.github.com/965229.js"></script>

Why you might ask? Because `attr_accessors_from_hash` isn't part of Ruby. The
method name is relatively self-explanatory but once you find the implementation
consider this. What the hell does that method do? I'd argue it's not very
readable and to be honest quite unnecessary.

## Dancing & Speaking

The board of directors made it very clear that in their opinion two of the most
important things each and everyone of our babes do is dance and speak. Both are
entirely dependent on the same factors, nationality. According to the
directors.

<script src="https://gist.github.com/965235.js"></script>

## New improved babes

With a `Nationality` class we can extract the nationality behaviour out of
`Babe` and delegate what a babe's likely to say or how she's going to dance to
her nationality improving code quality and perpetuating confusing stereotypes.

<script src="https://gist.github.com/965239.js"></script>

So that's it. Refactored and all round an improvement I think.

[Refactoring: Ruby Edition]: http://www.amazon.co.uk/gp/product/0321603508?ie=UTF8&tag=jameconrfinn-21&linkCode=as2&camp=1634&creative=19450&creativeASIN=0321603508
[Jay Fields]: http://blog.jayfields.com/
[David Heinemeier Hansson]: http://loudthinking.com)
[Adrian Holovaty]: http://www.holovaty.com/
[Snakes and Rubies]: http://video.google.com/videoplay?docid=1149552518153462279#
[Don Roberts]: http://www.refactory.com/people/don.html
