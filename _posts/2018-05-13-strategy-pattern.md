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
    def self.fly
      puts "I am flying"
    end
  end

  module CantFly
    def self.fly
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

## Asigning it by injection

We can still go further and apply the behavior on each instance by injection:

```ruby
module Flys
  module ItFlys
    def self.fly
      puts "I am flying"
    end
  end

  module CantFly
    def self.fly
      puts "I can't fly"
    end
   end
end

class Animal
  attr_accessor :fly_behavior

  def initialize(fly_behavior = Flys::CantFly)
    @fly_behavior = fly_behavior
  end

  def fly
    fly_behavior.fly
  end
end

class Lion < Animal
end

class Duck < Animal
end
```

```bash
irb> lion = Lion.new
irb> lion.fly
"I can't fly"
irb> duck = Duck.new(Flys::CantFly)
irb> duck.fly
"I can't fly"
irb> duck.fly_behavior = Flys::ItFlys
irb> duck.fly
"I am flying"
```

For the issue of flying animals, this last way of doing strategy design seems not the most appropiate. All ducks fly, so I personally prefer the previous solution for this case.

But, we have seen that in this last example we have the option to change the _fly_behavior_ of an instance. How could we do this with the code of the previous solution? I will do it in the next example using a **mixin** to change the flying behavior of an instance. That means, overwrite a **singleton method**. 

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
    return if @can_fly
    self.extend(Flys::ItFlys) if age >= DAYS_TO_FLY
  end
end
```

With this solution, an instance of a `Duck` will be able to fly at its seventh week of life, but not before.

