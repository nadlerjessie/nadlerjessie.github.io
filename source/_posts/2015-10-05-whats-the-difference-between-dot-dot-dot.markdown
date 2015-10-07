---
layout: post
title: "What's the difference between...?"
date: 2015-10-05 21:14:33 -0400
comments: true
categories: "Flatiron School"
---

As a new programmer I have frequently found myself asking the question, what’s the difference between these two methods? This is most common when it comes to enumerables. Here I’ll explain some differences between a few methods I find very useful. It is important to understand the differences so you can choose the best enumerable and create the most efficient code.

Before I dive in, it may be helpful to define an enumerable. Ruby-Doc defines the ```Enumerable``` module to provide collection classes with several traversal and searching methods, and with the ability to sort. 

Enumerable methods allow you to loop over all of the members of a given collection. Each member will be passed to a block. Some enumerables will simply verify the truthiness of a statement and others will return a modified argument. 

First let’s get some easy ones out of the way. What’s the difference between collect and map? Nothing, they’re aliases! What’s the difference between find and detect? Nothing!

Now that we’ve covered a few alias methods, let’s dig into some more complex comparisons. 

**Select vs Collect**
*Select* will iterate through each member and only return a given member if the information in the block is true. This returns a new array.
```ruby
array = [1, 2, 3, 4, 5]

array.select{ |num| num.even? }
=> [2, 4]
```

*Collect* will iterate through each member and return a new array with information based on the block. This could be an argument checking the truthiness or modifying the element, both shown below. 

```
array = [1, 2, 3, 4, 5]

array.collect{ |num| num.even? }
=> [false, true, false, true, false] 
```

The collect example above isn't that helpful to grab the even numbers. Collect can also modify the element that goes to the return, shown below.

``` 
array = [1, 2, 3, 4, 5]

array.collect{ |num| num/2 if num.even? }
=> [nil, 1, nil, 2, nil]
```

A the most apparent difference is that collect can modify an element before passing it to the return array. Select can only return the elements from the existing array. A less obvious, but important, difference is that collect will always return the same number of elements as the argument the method is called on. If the block in the collect statement is a logic statement, nil will be used if the block is false. Most often, you won’t want to carry around these nil values! 

**Select vs Find**
*Select*, as discussed above, will select all of the elements that cause the block to return true. *Find* will pull the first element that returns the block to be true. The big difference here may be obvious: select will return all of the elements that return true when passed to the block. A more nuanced difference is that find returns the element based on its current class. 

Starting with the example above, we see find returns the first instance that turns the block true.
```ruby
array = [1, 2, 3, 4, 5]

array.find{ |num| num.even? }
=> 2
```

Looking at the class of the return value tells us more about find's utility.

```ruby
hash_array = [ {1=>"one"}, {1 => "one", 2 => "two"} ]

useSelect = hash_array.select {|hash| hash if hash.keys.size > 1}
=> [ {1=>"one", 2=>"two"} ] #useSelect.class => Array

useFind =  hash_array.find {|hash| hash if hash.keys.size > 1}
=> {1=>"one", 2=>"two"}  #useFind.class => Hash
```

This may be useful if you don’t want the element you’re looking to pull to be put in array. Of course if you do in fact want all of the arguments that return true the elements can be extracted from the array using methods like first, last or by index! 
