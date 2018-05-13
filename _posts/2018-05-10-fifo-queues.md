---
layout: post
title:  "FIFO queues!"
date:   2018-05-10 17:14:47 +0200
categories: jekyll update
---

When I was a C developer we used very frequently **FIFO queues**. **FIFO** means _first input, first output_. This kind of queues have a fixed size. When the queue is full of elements, if a new element is enqueued, the first element enqueued has to fall out (has to be returned and removed from the queue).

**How would you implement this behavior with Ruby language?**

Here is how I would do it:

```ruby
## fifo_queue.rb

class FIFOQueue
  attr_accessor :size, :arr

  def self.[](*values)
    obj = self.new(values.size)
    obj.arr = values
    obj
  end

  def initialize(size)
    @size = size
    @arr  = Array.new
  end

  def push(element)
    arr.push(element)
    arr.shift if arr.length > size
  end
end

class Array
  def to_fifo
    FIFOQueue[*self]
  end
end
```

Adding the method `to_fifo` to the `Array` class permits us to create a new Array as we normally do and convert it a **FIFO queue**.

```sh
> irb -I . -r fifo_queue.rb
irb> ["a", 2, 3].to_fifo
=> #<FIFOQueue: @size=3, @arr=["a", 2, 3]>
irb> b = _
=> #<FIFOQueue: @size=3, @arr=["a", 2, 3]>
irb> b.push "b"
=> "a"
irb> b.arr
=> [2, 3, "b"]
```

