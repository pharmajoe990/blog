---
layout: post
title:  "Bubble sort algorithm"
date:   2019-05-09 12:54:00 +1100
categories: algorithms development ruby
---

### Update

_I've been meaning to play around with some algorithms for a while. I have some random code lying around that could do with a brain dump, so here goes._

## Bubble sort you say?

Bubble sort is a sorting algorithm that can be applied to any list dataset where the elements can be compared or swapped based on a boolean comparison operator. It is a relatively expensive implementation, compared with other algorithms for sorting data, due to the requirement to iterate and compare the entire dataset multiple times. Because of this, it is not really useful in any real-world implementation for a Production Application. It is relatively easy to understand and implement however.

## Down to business

The basic logic operates as follows;

* Compare the first 2 elements of the array and;
    * if the first element is larger than the next, swap them
    * Otherwise leave as is
    * Move to the next element
* Do this until the end of the array is reached
* If there was at least 1 swap carried out for the array move to the start of the array and start again
* When there are no swaps carried out whilst iterating the array, the sort is completed

Here is the implementation (Ruby)

```ruby
#!/usr/bin/env ruby

def bubble_sort(numbers)
  start_time = Time.now
  num_iterations = 0
  begin
    swapped = false
    numbers.each_with_index do |element,index|
      comp_index = index
      other_comp_index = index + 1
      if other_comp_index < numbers.size
        if numbers[comp_index] > numbers[other_comp_index]
          temp = numbers[other_comp_index]
          numbers[other_comp_index] = numbers[comp_index]
          numbers[comp_index] = temp
          swapped = true
        end
      end
    end
    num_iterations += 1
  end while swapped
  end_time = Time.now
  exec_time = end_time - start_time
  {num_iterations: num_iterations, execution_time: exec_time }
end

def random_integer_array(max_size, max_int)
  array = []
  array_length = (max_size * rand).to_i
  array_length.times { array.push((rand * max_int).to_i) }
  array
end

def exec(max_array_size, max_num)
  random_numbers = random_integer_array(max_array_size, max_num)
  stats = bubble_sort(random_numbers)
  puts "|#{stats[:num_iterations]}|#{stats[:execution_time]}|#{random_numbers.length}|"
end

max_array_size = 10_000
max_num = 10_000_000

25.times do
  exec(max_array_size, max_num)
end
```

## Operational notes

This implementation will kindly print out some useful information on the algorithm for us. First we set the desired maximum array length, and maximum number to populate the array with. This will give us a randomly sized array of random numbers. We also get the total number of iterations it took to carry out the sort, as well as the run time in seconds. For the above parameters, we get the following stats:

|Iterations|Time|Length|
|-|-|
|4740|2.167903|4760|
|7558|5.732699|7688|
|8609|7.606905|8840|
|6941|4.932411|7037|
|423|0.018368|432|
|1030|0.113208|1064|
|6159|3.931894|6278|
|6256|3.969807|6288|
|9050|8.477311|9217|
|4853|2.419822|5048|
|6424|4.17767|6575|
|6890|4.836892|7084|
|6780|4.686801|6908|
|9299|8.742375|9455|
|2784|0.81261|2857|
|8115|6.611018|8218|
|6300|3.942655|6339|
|886|0.082821|923|
|1084|0.117848|1129|
|8178|6.924275|8438|
|7655|5.982082|7803|
|7463|5.406711|7541|
|2015|0.41954|2058|
|712|0.05402|746|
|4190|1.744643|4250|
 
 So, based on the length of the Array being sorted, we can see it directly affects the sort time. Depending on the spread of values, the iterations it takes to sort can take up to the entire length of the array, which for a sizable dataset can be resource hungry. If the comparison operation (to identify if a swap is required) is relatively expensive, depending on the datatype, this could add up to a resource-hungry, expensive sort.
 
 Having said that, this approach is a good start to investigating sorting algorithms due to it's relative simplicity in operation and implementation.
