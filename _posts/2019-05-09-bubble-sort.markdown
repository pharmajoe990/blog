---
layout: post
title:  "Bubble sort algorithm"
date:   2019-05-09 12:54:00 +1100
categories: algorithms development
---

### Update

_I've been meaning to play around with some algorithms for a while. I have some random code lying around that could do with a brain dump, so here goes._

## Who cares about algorithms anyway?

## Bubble sort you say?

## Down to business

Here is the implementation

```ruby
#!/usr/bin/env ruby

def bubble_sort(numbers)
  begin
    swapped = false # Flag to indicate sort is complete                                         
    numbers.each_with_index do |element,index|
      comp_index = index # first index of array for comparison
      other_comp_index = index + 1 # Second index for comparison to above
      if other_comp_index < numbers.size
        if numbers[comp_index] > numbers[other_comp_index] # Swap? 
          temp = numbers[other_comp_index]
          numbers[other_comp_index] = numbers[comp_index]
          numbers[comp_index] = temp
          swapped = true
        end
      end
    end
  end while swapped
  numbers
end

def random_integer_array(size, max_int)
  array = []
  size.times { array.push((rand * max_int).to_i) }
  array
end

array_size = 100
numbers = (1..array_size).to_a.shuffle
random_numbers = random_integer_array(array_size, 1000)

pp numbers
pp bubble_sort(random_numbers)

```

## Pros/Cons

* Expensive in comparison to other sorting algorithms
* Relatively easy to understand