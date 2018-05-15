---
layout: post
title:  "Stub methods like Rspec"
date:   2018-05-15 16:40:00 +0200
categories: jekyll update
permalink: /stub-methods-like-rspec/
---


Could you recognise what is it doing next code?

```ruby
let(:fran_user) { User.new }

before do
 allow(fran_user).to receive(:nickname).and_return("martinezcoder")
end

it { expect(fran_user.hi).to return("Hi, this is martinezcoder!")}
```

Sure! This is a call to stub a method with [**Rspec**](https://relishapp.com/rspec/rspec-mocks/docs/working-with-legacy-code/any-instance). But, 

* **Could you do the same without using Rspec?**
* **Do you know how to stub the method of a single instance?**

In this case, considering that `fran_user` is an instance of `User`, I am suposse to have a `User` class including the method `hi` and `nickname`. When running the test, if the instance `fran_user` uses the method `hi`, it will respond _"Hi, this is martinezcoder!"_, instead of the returned value defined in the method.

```ruby

class User
  def nickname
    "me"
  end

  def hi
    "Hi, this is #{nickname}!"
  end
end

fran_user = User.new
puts fran_user.hi
```

Last code will puts: 

```sh
=> "Hi, this is me!"
```

### How to stub the instance of User without using Rspec?

Stub the `nickname` method of the `fran_user` instance:

```ruby
fran_user = User.new
puts fran_user.hi

class << fran_user
  def nickname
    "martinezcoder"
  end
end

puts fran_user.hi
```

Last code will puts:

```sh
=> "Hi, this is me!"
=> "Hi, this is martinezcoder!"
```

So you now know how to stub an instance method without the dependency of **Rspec**!


