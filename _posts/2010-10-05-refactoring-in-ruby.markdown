---
layout: post
title: Refactoring in Ruby
---

Writing clean code that's easily understandable is something I think the Ruby
community really appreciate. Ruby gives us one hundred and one ways to do
something but there's normally a beautiful, elegant way and other less
impressive ways.

Here's some pretty ugly code that isn't very DRY and isn't very readable. Before
we start we make sure we've got full test coverage so we don't break any of our
awesome babe logic.

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

---

## Initialisation

Let's start with the #initialize method. It takes a whole lot of arguments, in
a very specific order, which makes it quite difficult to work with.

{% highlight ruby %}
def initialize(name, attributes = {})
  @name = name
  attributes.each do |attribute, value|
    instance_variable_set("@#{attribute}", value)
  end
end
{% endhighlight %}

This will work but isn't all that readable and what's worse, there's absolutely
no validation of a babe's attributes!

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

Use of `VALID_ATTRIBUTES` means we ignore invalid arguments but doesn't account
for typos or alike as our babes will ignore all number of attributes they just
don't care about without complaining one little bit.

Pulling in `ActiveSupport`'s Hash extensions give us the `#assert_valid_keys`
method This is where we maybe get a bit lazy and yield to the temptation to use
some unnecessary metaprogramming.

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

Wait, wait, wait! Adding an instance method to all objects?! No thanks. We could
avoid this using a module…

{% highlight ruby %}
require 'active_support/core_ext/hash'

module MagicAttributeAssignment
  def attr_accessors_from_hash(hash)
    hash.each do |name, value|
      self.class.send(:attr_accessor, name)
      self.send("#{name}=", value)
    end
  end
end

class Babe
  include MagicAttributeAssignment

  VALID_ATTRIBUTES = [:hair_color, :weight, :hotness, :nationality]

  def initialize(name, attributes = {})
    @name = name
    attributes.assert_valid_keys(*VALID_ATTRIBUTES)
    attr_accessors_from_hash(attributes)
  end

  # ...
end
{% endhighlight %}

…but I think this is too magic and frankly makes our babes less understandable.
That's obviously not what we want.

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

This is my personal favourite. Self-documenting and finger-fail proof.

---

## Dancing & Speaking

Two of the most important things each and everyone of our babes have to be able
to do. But both are entirely dependent on the same factors, nationality.

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

  def find(place)
    all.detect { |nationality| nationality.name == place }
  end

  alias :dance :dance_move
  alias :speak :speech
end

class Babe
  extend Forwardable

  VALID_ATTRIBUTES = [:hair_color, :weight, :hotness, :nationality]

  def_delegators :nationality, :dance, :speak
  attr_accessor *VALID_ATTRIBUTES

  def initialize(name, attributes = {})
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
    weight < 100
  end

  def beautiful?
    hotness > 2
  end

  def in_my_league?
    fit? ^ beautiful?
  end
end
{% endhighlight %}

So that's it. Refactored and all round an improvement I think.
