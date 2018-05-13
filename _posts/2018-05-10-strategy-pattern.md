---
layout: post
title:  "Strategy pattern and singleton methods"
date:   2018-05-13 18:00:00 +0200
categories: jekyll update
---

Imagine that you have a collection of objects representing _animals_. Some of them can fly and some others not. You would like to include the method `fly` in all them, although not all them can fly, so you go with the **Template Method Design Pattern**. 


```ruby
class Animal
  def fly
    raise NotImplementedError
  end
end

class Lion < Animal
  def fly
    "I can't fly!"
  end
end

class Duck < Animal
  def fly
    "I am flying!"
  end
end
```


Depending on the class, you decide whether it can fly or not. But this is not very clever, because we will be creating a lot of duplicate code. We will have as many `fly` methods as animals we create, and all of them will have the content repeated again and again.

We can improve the architecture of the code with **Strategy Design Pattern**.

## Strategy Design Pattern

We have two options (**strategies**) to decide to include on each animal: _it can fly_ or _it can not_. So we create a module containing both strategies:

```ruby
module Flys
  module ItFlys
    def fly
      puts "I am flying"
    end
  end

  module CantFly
    def fly
      puts "I can't fly"
    end
   end
end
```

We decide that animals can't fly as a default. And we include the `CanFly` strategy in the animals that can fly:

```ruby
class Animal
  include Flys::CantFly
end

class Lion < Animal
end

class Duck < Animal
  include Flys::ItFlys
end
```

```bash
irb> lion = Lion.new
irb> lion.fly
"I can't fly"
irb> duck = Duck.new
irb> duck.fly
"I am flying"
```


## Singleton methods

But wait, a duck with 1 day of life is not able to fly, so, can a duck fly? Well, it depends. Next code will demostrate how to change this behaviour depending on the value of the instance variable `age`. 

When the duck is old enough, it can fly!


```ruby
class Animal
  include Flys::CantFly

  attr_accessor :age

  def initialize(age=0)
    @age = age
  end
end

class Lion < Animal
end

class Duck < Animal
  DAYS_TO_FLY = 50

  def initialize(age=0)
    super(age)
    @can_fly = false
    try_to_fly
  end

  def age=(days)
    super(days)
    try_to_fly
  end

  private

  def try_to_fly
    if age >= DAYS_TO_FLY && !@can_fly
      self.extend(Flys::ItFlys)
      @can_fly = true
    end
  end
end
```

With this solution, an instance of a `Duck` will be able to fly at its seventh week of life, but not before.
