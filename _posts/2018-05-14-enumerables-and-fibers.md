---
layout: post
title:  "Fibers, Procs and Enumerables"
date:   2018-05-13 18:00:00 +0200
categories: jekyll update
---

Last day I spend some hours investigating a bit deeply about `Enumerable`. It is that kind of stuff that shows the magic of the **Ruby** language. Once you have the whole control of enumerables, you feel the power of being a Ruby developer.

**Fibers**, **Procs** and **Enumerables** are different things in **Ruby**. But I will resolve the very traditional example of showing prime numbers with each one of them.

We will define a module with the common code to find the prime numbers:

```ruby
module PrimeFinder
  private

  def find_next_prime(x)
    x += 1 while !prime?(x)
    x
  end

  def prime?(x)
    i = 2
    while i <= Math.sqrt(x)
      return false if x % i == 0
      i += 1
    end
    true
  end
end
```

## Proc

`Proc` saves the last state of the `x` variable, so in every `call` it will return the value of the next prime number:

```ruby

class Prime
  include PrimeFinder

  def proc
    x = 1
    Proc.new do
      x = find_next_prime(x+1)
    end
  end
end

```

```sh
irb> p = Prime.new.proc
=> #<Proc:test/fiber.rb>
irb> p.call
=> 2
irb> p.call
=> 3
irb> p.call
=> 5
irb> p.call
=> 7
irb> 5.times { puts p.call }
11
13
17
19
23
=> 5
```

`Proc` objects picks up the surrounding environment. Any variables that are visible when a `Proc` is created remain visible inside the `Proc` when it is run. That's why `x` inside the `Proc` object is the same as the `x` outside. 

Also, a `Proc` always returns the last value computed in the code block. This is because `Proc` objects have a lot in common with methods, so the last expression in the block will be the returned value of `call`.

In summary, this solution does the trick, but it doesn't seem the best solution for this issue.

## Fiber

[Fibers](https://ruby-doc.org/core-2.4.1/Fiber.html) are light weight primitives in the Ruby standard library which can be paused, resumed and scheduled manually. Fibers are commonly used in combination of concurrence processes, because they simplify asynchronous code.

In this case, it will just return _next_ prime number and _pause_ the loop.


```ruby

class Prime
  include PrimeFinder

  def fiber
    Fiber.new do
      x = 1
      loop do
        Fiber.yield x = find_next_prime(x+1)
      end
    end
  end
end

```

```sh
irb> f = Prime.new.fiber
=> #<Fiber:(irb):3 (created)>
irb> f2 =  Prime.new.fiber
=> #<Fiber:(irb):3 (created)>
irb> f.resume
=> 2
irb> f.resume
=> 3
irb> 3.times { puts f.resume }
5
7
11
=> 3
irb> 3.times { puts f2.resume }
2
3
5
=> 3
```

Basically, `Fiber#yield` returns control back to the context that resumed the Fiber and returns the value which was passed to `Fiber#resume`. Each defined fiber, will control its own independent state. That's why it is useful in concurrence executions.


## Enumerable

### Option A

The traditional way with the `include Enumerable` and defining an `each` method. This way, we can use all the methods for enumerables like `take`, `first`, etc.

```ruby
class Prime
  include PrimeFinder
  include Enumerable

  def each(&block)
    x = 1
    loop do
      yield x = find_next_prime(x+1)
    end
  end
end
```

Note that using `take` and `first` of **Enumerable** always start by the first prime number `2`. But if we use `to_enum`, the result saves the last state of the loop.

```sh
irb> p = Prime.new
=> #<Prime:>
irb> p.take(3)
=> [2, 3, 5]
irb> p.first(3)
=> [2, 3, 5]
irb> e = p.to_enum
=> #<Enumerator: #<Prime:>:each>
irb> e.next
=> 2
irb> e.next
=> 3
irb> e.next
=> 5
irb> e.next
=> 7
irb> e.next
=> 11
```

### Option B

Instead of using the `include Enumerable`, we define an enumerable for an specific method. Is the same as using `to_enum` in the last example.

```ruby
class Prime
  include PrimeFinder

  def enum
    x = 1
    return enum_for(:enum) if !block_given?

    loop do
      yield x = find_next_prime(x+1)
    end
  end
end
```


```sh
irb> p = Prime.new.enum
=> #<Enumerator: #<Prime:>:enum>
irb> p.next
=> 2
irb> p.next
=> 3
irb> p.next
=> 5
irb> p.next
=> 7
irb> p.take(3)
=> [2, 3, 5]
irb> p.first(3)
=> [2, 3, 5]
irb> p.next
=> 11
```

Note how the last `p.next` has continue the list showed before in the last call to `p.next`. But in the middle, we called `p.take` and `p.first` and the index of the state was not altered.


## Conclusion

**Ruby** offers us different ways to do the same. In this case, the use of **Enumerable** is the best approach.

I have been playing with `Fiber` and `Proc` to achieve similar results, but their purpose are not to resolve this kind of issues. Also, I investigate about `Fiber` and I found that **Matz** [proposed](https://bugs.ruby-lang.org/issues/8572) to include the `Enumerable` to `Fiber` by default. The proposal was finally rejected because they didn't find any reasonable use case.



