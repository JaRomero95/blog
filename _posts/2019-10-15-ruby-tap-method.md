---
title: The tap method in Ruby
author: Juan Antonio Romero Molero
date: 2019-09-10 19:00:00 +0200
categories: [Backend]
tags: [ruby, clean code]
---

## Introduction

Ruby's tap method is as simple as it is effective, although not many use it.

This method belongs to the `Object` class and basically what it does when we invoke it on an instance is send self to the block we pass to it and finally returns the same instance.

It's useful when we need a method to return the instance itself instead of the last operation result, and in this way to be able to concatenate with other similar methods or simply avoid referencing it in an auxiliary variable.

## How to use it

Let's see an example to understand how it works. We have the following Ruby class:

```ruby
class Hobbit
  def eat
    puts "#{name} eats a whole lembas bread."
    self.hungry = false
  end

  def smoke
    puts "#{name} takes a break to smoke Longbottom Leaf."
    self.stressed = false
  end
end
```

To test it, we are going to instantiate an object of this class and invoke its methods.

```ruby
hobbit = Hobbit.new('Pippin')

hobbit.eat
hobbit.smoke
```

It's not bad, but it'll be better if we could call both methods in a row. We are going to change these methods so they return the instance at the end of each one to be able to do *method chaining*.

```ruby
class Hobbit
  def eat
    puts "#{name} eats a whole lembas bread."
    self.hungry = false
    self
  end

  def smoke
    puts "#{name} takes a break to smoke Longbottom Leaf."
    self.stressed = false
    self
  end
end
```

Now, we can do *method chaining*:

```ruby
hobbit = Hobbit.new('Pippin')

hobbit.eat.smoke
```

We have achieved what we wanted, but returning `self` explicitly at the end of each method seems a bit forced. This is where the `tap` method comes to the rescue to make our code better express itself. Let's change our methods again:

```ruby
class Hobbit
  def eat
    tap do |hobbit|
      puts "#{hobbit.name} eats a whole lembas bread."
      hobbit.hungry = false
    end
  end

  def smoke
    tap do |hobbit|
      puts "#{hobbit.name} takes a break to smoke Longbottom Leaf."
      hobbit.stressed = false
    end
  end
end
```

Now we keep the ability to do *method chaining* but with a little more syntax sugar.

It's true that the example is too basic to see the usefulness of this method because it was even shorter in the previous form, also we don't need to use the variable that the method receives. Let's look at a more useful example for the real world:

```ruby
class CharacterFactory
  def create_sample_hobbit
    character = Humanoid.new
    character.height = 90
    character.weight = 35_000
    features = %i[hairy_feet good_aim]
    character.add_features(features)
    self
  end
end
```

And now we are going to refactor this code using the tap method. Let's see:


```ruby
class CharacterFactory
  def create_sample_hobbit
    Humanoid.new.tap do |character|
      character.height = 90
      character.weight = 35_000
      features = %i[hairy_feet good_aim]
      character.add_features(features)
    end
  end
end
```

Grouping actions in the tap block is useful for clarifying intentions in our code, so the previous example will give us a slight improvement in readability.

## Conclusion

That's all! Personally, I like to use these details that Ruby offers to try to make my code more expressive and easy to read, but these are just little things that can be useful in very specific situations, and at the end of the day, it is up to you to choose a style or other.