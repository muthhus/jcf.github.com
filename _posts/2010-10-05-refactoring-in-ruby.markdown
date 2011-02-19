---
layout: post
title: Refactoring in Ruby
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

{% highlight ruby %}
class Babe
  def initialize(name, hair_color, weight, hotness, nationality)
    @name = name
    @hair_color = hair_color
    @weight = weight
    @hotness = hotness
    @nationality = nationality
  end

  def dance
    if nationality == 'American'
      return 'Grind'
    elsif nationality == 'British'
      return 'Wiggle'
    elsif type == 'German'
      return 'Strut'
    else
      return 'Bounce'
    end
  end

  def speak
    if nationality == 'American'
      return 'Totally'
    elsif nationality == 'British'
      return 'Woteva'
    elsif type == 'German'
      return 'Ja'
    else
      return 'Hi'
    end
  end

  def fit?
    @weight < 100
  end

  def beautiful?
    @hotness > 2
  end

  def in_my_league?
    if fit?
      if beautiful?
        return false
      else
        return true
      end
    end
    false
  end
end
{% endhighlight %}

Now we're doing a code review and looking at how a babe works. There's some
repetition in the `#dance` and `#speak` methods and the `#in_my_league?` method
has grown in to something quite difficult to understand.

We're working with Ruby, which means we can do almost anything we want to a babe
but let's remember what DHH said, there's always a beautiful way to do something
and that's what we want. Beautiful babes.

## Initialisation

Let's start with the `#initialize` method. It takes a whole lot of arguments, in
a very specific order, which makes it quite difficult to work with.

{% highlight ruby %}
def initialize(name, attributes = {})
  @name = name
  attributes.each do |attribute, value|
    instance_variable_set("@#{attribute}", value)
  end
end
{% endhighlight %}

Using a hash of optional attributes gives us named arguments. Much nicer when
creating a new babe but not ideal when you don't aren't sure which attributes
in a babe matter to the boss!

So we could define valid attributes and only extract those using `#slice`.

{% highlight ruby %}
VALID_ATTRIBUTES = [:hair_color, :weight, :hotness, :nationality]

def initialize(name, attributes = {})
  @name = name
  extract_attributes(attributes)
end

private

def extract_attributes(attributes)
  attributes.slice(*VALID_ATTRIBUTES).each do |name, value|
    instance_variable_set("@#{name}", value)
  end
end
{% endhighlight %}

Better but not beautiful, yet. Named arguments are great but not validating them
doesn't feel right in my opinion.

A number of methods in Rails use `#assert_valid_keys` to ensure only valid
options are passed to a method and we can do the same.

{% highlight ruby %}
require 'active_support/core_ext/hash'

class Babe
  VALID_ATTRIBUTES = [:hair_color, :weight, :hotness, :nationality]

  attr_accessor *VALID_ATTRIBUTES

  def initialize(name, attributes = {})
    @name = name
    attributes.assert_valid_keys(*VALID_ATTRIBUTES)
    @hair_color  = attributes[:hair_color]
    @weight      = attributes[:weight]
    @hotness     = attributes[:hotness]
    @nationality = attributes[:nationality]
  end

  # ...
end
{% endhighlight %}

### Bad Metamonkey!

Refactoring does not mean "show me your Ruby foo!" Temptation to add crazy
methods like I do below is often best resisted.

{% highlight ruby %}
require 'active_support/core_ext/hash'

class Object
  def attr_accessors_from_hash(hash)
    hash.each do |name, value|
      self.class.send(:attr_accessor, name)
      self.send("#{name}=", value)
    end
  end
end

class Babe
  VALID_ATTRIBUTES = [:hair_color, :weight, :hotness, :nationality]

  def initialize(name, attributes = {})
    @name = name
    attributes.assert_valid_keys(*VALID_ATTRIBUTES)
    attr_accessors_from_hash(attributes)
  end

  # ...
end
{% endhighlight %}

Why you might ask? Because `attr_accessors_from_hash` isn't part of Ruby. The
method name is relatively self-explanatory but once you find the implementation
consider this. What the hell does that method do? I'd argue it's not very
readable and to be honest quite unnecessary.

## Dancing & Speaking

The board of directors made it very clear that in their opinion two of the most
important things each and everyone of our babes do is dance and speak. Both are
entirely dependent on the same factors, nationality. According to the
directors.

{% highlight ruby %}
class Babe
  SKILLS = {
    'American' => %w(Grind Totally),
    'British' => %w(Wiggle Woteva),
    'German' => %w(Strut Ja),
  }
  SKILLS.default = %w(Bounce Hi)

  # ...

  def dance
    SKILLS[nationality].first
  end

  def speak
    SKILLS[nationality].last
  end

  # ...
end
{% endhighlight %}

This is much more DRY but calling #first and #last after a hash lookup? We could
opt for two hashes…

{% highlight ruby %}
class Babe
  DANCE_MOVES = {
    'American' => 'Grind',
    'British'  => 'Wiggle',
    'German'   => 'Strut'
  }
  DANCE_MOVES.default = 'Bounce'

  SOUNDS = {
    'American' => 'Totally',
    'British'  => 'Woteva',
    'German'   => 'Ja'
  }
  SOUNDS.default = 'Hi'

  def dance
    DANCE_MOVES[nationality]
  end

  def speak
    SOUNDS[nationality]
  end
end
{% endhighlight %}

You might think of subclassing babes…

{% highlight ruby %}
class Babe
  # ...

  def dance
    'Bounce'
  end

  def speak
    'Hi'
  end

  # ...
end

class AmericanBabe < Babe
  def dance
    'Grind'
  end

  def speak
    'Totally'
  end
end
{% endhighlight %}

…but we're subclassing them because they have a different nationality and the
behaviour that's specific to a nationality leads me to think maybe we should
have a `Nationality` object, which has an association with a babe.

{% highlight ruby %}
class Nationality < Struct.new(:name, :dance_move, :speech)
  def self.all
    @nationalities ||= [
      Nationality.new('American', 'Grind', 'Totally'),
      Nationality.new('British', 'Wiggle', 'Woteva'),
      Nationality.new('German', 'Strut', 'Ja'),
    ]
  end

  def self.default
    @default ||= Nationality.new('Unknown', 'Bounce', 'Hi')
  end

  def self.find(place)
    all.detect { |nationality| nationality.name == place }
  end

  alias :dance :dance_move
  alias :speak :speech
end

class Babe
  extend Forwardable

  def_delegators :nationality, :dance, :speak
  # ...

  def initialize(name, attributes = {})
    # ...
    self.nationality = attributes[:nationality]
    # ...
  end

  def nationality
    @nationality || Nationality.default
  end

  def nationality=(place)
    @nationality = Nationality.find(place)
  end

  # ...
end
{% endhighlight %}

## Fit and beautiful?!

Conditionals should be readable and easy to follow. That's where extract query
in to method will often help, but sometimes that's not enough. Just look at the
state of the `#in_my_league?` method!

{% highlight ruby %}
def in_my_league?
  if fit?
    if beautiful?
      return false
    else
      return true
    end
  end
  false
end
{% endhighlight %}

The way that method is written is incredibly unreadable and ultimately all it's
doing when you manage to wrap your head around the logic implemented is an XOR.

{% highlight ruby %}
def in_my_league?
  fit? ^ beautiful?
end
{% endhighlight %}

Really? That simple? Surely not… Well, you might be surprised to hear it is.

## New improved babes

With a `Nationality` class we can extract the nationality behaviour out of
`Babe` and delegate what a babe's likely to say or how she's going to dance to
her nationality improving code quality and perpetuating confusing stereotypes.

{% highlight ruby %}
require 'active_support/core_ext/hash'

class Nationality < Struct.new(:name, :dance_move, :speech)
  class << self
    def all
      @nationalities ||= [
        Nationality.new('American', 'Grind', 'Totally'),
        Nationality.new('British', 'Wiggle', 'Woteva'),
        Nationality.new('German', 'Strut', 'Ja'),
      ]
    end

    def default
      @default ||= Nationality.new('Unknown', 'Bounce', 'Hi')
    end

    def find(place)
      all.detect { |nationality| nationality.name == place }
    end
  end

  alias :dance :dance_move
  alias :speak :speech
end

class Babe
  extend Forwardable

  VALID_ATTRIBUTES = [:hair_color, :weight, :hotness, :nationality]

  def_delegators :nationality, :dance, :speak
  attr_accessor *VALID_ATTRIBUTES

  def initialize(name, weight, hotness, attributes = {})
    attributes.assert_valid_keys(*VALID_ATTRIBUTES)

    self.name        = name
    self.hair_color  = attributes[:hair_color]
    self.weight      = attributes[:weight]
    self.hotness     = attributes[:hotness]
    self.nationality = attributes[:nationality]
  end

  def nationality
    @nationality || Nationality.default
  end

  def nationality=(place)
    @nationality = Nationality.find(place)
  end

  def fit?
    weight.try(:<, 100)
  end

  def beautiful?
    hotness.try(:>, 2)
  end

  def in_my_league?
    fit? ^ beautiful?
  end
end
{% endhighlight %}

So that's it. Refactored and all round an improvement I think.

[Refactoring: Ruby Edition]: http://www.amazon.co.uk/gp/product/0321603508?ie=UTF8&tag=jameconrfinn-21&linkCode=as2&camp=1634&creative=19450&creativeASIN=0321603508
[Jay Fields]: http://blog.jayfields.com/
[David Heinemeier Hansson]: http://loudthinking.com)
[Adrian Holovaty]: http://www.holovaty.com/
[Snakes and Rubies]: http://video.google.com/videoplay?docid=1149552518153462279#
[Don Roberts]: http://www.refactory.com/people/don.html
