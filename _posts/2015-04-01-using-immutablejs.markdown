---
layout: post
title:  "Using ImmutableJS"
date:   2015-04-01 09:50:45
categories: immutablejs react
---

If you don't know what ImmutableJS is, it's written by facebook and can be used to make very fast react applications.
[Check out this video](https://www.youtube.com/watch?v=I7IdS-PbEgI) for a better explanation than I could give you 

ImmutableJS is an implementation of persistent data structures and is extremely powerful, so long as you understand and use the data structures properly.
However, understanding (much less using) the ImmutableJS data structures are extremely hard to use because the docs are unhelpful and confusing.
This post is to address which data structures you should use and you should use and when to use them.

# Navigation
* [Watch Out](#watchout)
* [Performance](#performance)
* [Key terms](#keyterms)
* [Data Structures](#datastructures)

## <a name="watchout">Watch Out (Some things that got me)</a>

1. toArray(), toJS(), and toObject() do not return a new thing, they convert the immutable object in place. Which is decieving
  because these are the only things like that (that I have found).


## <a name="performance" >Some quick perfomance considerations.</a>

ImmutableJS does a shallow copy of an object every time you mutate it, which can have massive perforamance implications.
If you don't consider your data structers while working with them you can freeze up your page really easily. 
[if you run performance test](http://jsperf.com/with-out-mutatable) it will be clear that one way of doing things is 
extremely slow, and one is fast, but neither are nearly as fast as plain javascript.

###Lets break down the code

Here is our setup
{% highlight javascript %}
// Variable Declarations
var arr = Immutable.List(),
  jsArr = [],
  testSize = 100000;
{% endhighlight %}

In this example we are creating an array filled with 0 - 99999.
Since we are all famialiar with javascript. Here is how most of us would do it.

--------------------------------------------------------------------------------------------------------------
This is the regular way, and is, comparatively, blazingly fast (230 per second).

{% highlight javascript %}
for (var i = 0; i < testSize; i++) {
  jsArr.push(i);
}
{% endhighlight %}

--------------------------------------------------------------------------------------------------------------

Here is the wrong way to do it, and it's painfully slow (11 per second).

{% highlight javascript %}
for (var i = 0; i < testSize; i++) {
  imList = imList.push(i);
}
{% endhighlight %}

But why is it so slow? Because every time you push you are also copying the entire list!

--------------------------------------------------------------------------------------------------------------

_So what is the answer? Is there a better way._
My first thought was to create the array at blazing speed, then create the Immutable List and be on my merry way!

Reality didn't match my expectation however (27 per second)

{% highlight javascript %}
for (var i = 0; i < testSize; i++) {
  jsArr.push(i);
}
imList = Immutable.List(jsArr); 
{% endhighlight %}

Why didn't that work? Well transforming a giant array into a different datastructure isn't as performant as I would have hoped.

--------------------------------------------------------------------------------------------------------------
Finally I stubmled on, what I consider, to be the best answer.

There is a function called `withMutations` which gives you a mutatable version as a callback, which you can mutate mutliple times
without worrying about shallow copies happening every time (49 per second).

{% highlight javascript %}
imList = imList.withMutations(function(mutatable) {
  for (var i = 0; i < testSize; i++) {
    mutatable.push(testSize);
  }
  return mutatable;
});
{% endhighlight %}

--------------------------------------------------------------------------------------------------------------

There is one more answer, however there are some drawbacks (15,879,672 per second). 

{% highlight javascript %}
var imRange = Immutable.Range(0, testSize);
{% endhighlight %}


_Uhm what? 15,879,672 per second? 75979 times faster than a javascript array? What is this black magic?_

Range is a version of Seq (sequence) which is lazy, and won't do any work until you need to use it. So if you
can think of a case where you **might** need an array of 999999 elements, then this is perfect for you. I,
however, can't think of such a case. While this seems like a cool thing to have access too, I haven't found any practical uses.

--------------------------------------------------------------------------------------------------------------
## <a name="keyterms">Some Key terms</a>

1. keypath: Throughout these immutablejs docs, you will see this term. It is just a path to access nested properties. 
  EX: `'steve.pants.leftPocket.wallet.creditCardSlots.1'`

2. [amortized](http://en.wikipedia.org/wiki/Amortized_analysis): If a data structure is "amortized O(log32 N)"
  It means that in practical applications you can expect the data structure to perform specified operations at that 
  [Order of magnitude](http://en.wikipedia.org/wiki/Orders_of_magnitude_%28data%29).

3. [stable](http://en.wikipedia.org/wiki/Sorting_algorithm#Stability): A stable sorting algorithm means that in the case two items are the same, 
    the first item given to the sorting algorithm will come out first in the sorted structure.


## <a name="datastructures">Data Structures</a>
* [Map](#Map)
* [List](#List)
* [OrderedMap](#OrderedMap)
* [Set](#Set)
* [OrderedSet](#OrderedSet)
* [Stack](#Stack)
* [Seq](#Seq)
* [KeyedSeq](#KeyedSeq)
* [SetSeq](#SetSeq)

--------------------------------------------------------------------------------------------------------------

### [Map](https://facebook.github.io/immutable-js/docs/#/Map) <a name="Map"></a>
  Equivalent to a regular javascript object and can be used in similar places, the disadvantage is that you need to use getters and setters.
  You can iterate over these, but the order is not guaranteed.

  An alternative to this is a Record, which you don't need to use getters for.

--------------------------------------------------------------------------------------------------------------

### [List](https://facebook.github.io/immutable-js/docs/#/List) <a name="List"></a>

  Most like a javascript array in that its contents are kept in order. However, you might find this less useful than a javascript array because it's contents are not mutatable.
  

  _Consider the following scenario:_

  >You are creating a list of tasks. Seems like a great use case for a list. It is a list after all. 
  >You create the basic web app allowing you too create a basic task, and each task is added to the end of the list. Perfect, publish the app.
  >You are a great success and soon enough people start asking for features. The first thing they ask for is that they can edit previous tasks.
  >This would be easy to implement traditionally, however you can't simply grab the task object you associated with the one they want to click on and change it anymore.
  >You need to edit the task **and** replace it in your list of tasks. Now you have to do a search through the list, find the task, and replace it with a new edited one.
  >Suddenly a simple mutation becomes an O(n) operation and five or six more lines of code. Perhaps you should consider an OrderedMap instead._

--------------------------------------------------------------------------------------------------------------

### [Ordered Map](https://facebook.github.io/immutable-js/docs/#/OrderedMap)  <a name="OrderedMap"></a>
  
  An ordered map is like a combination of a javascript object and an array. You set things with an index, and order in which is you set things is guaranteed.

  _While discussing List, we came to the realization that a List would no longer work for us the way an array did, Ordered Maps can fill the gap._

  >Instead of using a list, use an ordered map by creating a unique index for each new task. 
  >Preferably use an index that will sort to be the same order you are looking for. 
  >You could just use the ordered map size as the index or something like a UNIX time stamp.

  >The advantage of doing this is that you can find the object again later so that you can edit/replace it in the Ordered Map later.
  >One caveat is that you also need to keep the index on the object in the Ordered Map so you can tell where the object came from later.

--------------------------------------------------------------------------------------------------------------

### [Set](https://facebook.github.io/immutable-js/docs/#/Set) - _[Example](http://codepen.io/ericwooley/pen/bNJRZJ)_ <a name="Set"></a>

  A set is like a list but prevents the same entry from occurring twice. Which sounds useful, until you remember that every mutation returns a new object. 
  This is perfect for keeping a list of unique primitive values. 

  _Can't we use this instead of an ordered map in our task list?_

  >It's tempting to assume that we could use this as a quick way to `replace` things in our task list. 
  >The thing to remember is that ImmutableJS objects always return a new copied object so the set would just add any mutation of a task you created,
  >and would be of no help.

--------------------------------------------------------------------------------------------------------------

### [Ordered Set](http://facebook.github.io/immutable-js/docs/#/OrderedSet) <a name="OrderedSet"></a>
  Just like a set, but will preserve the order in which things are added, so that you can iterate over in the same way later. 
  Most the time set keeps the order also, but it's not guaranteed. 
  An ordered set guarantees the order at a memory cost.

--------------------------------------------------------------------------------------------------------------

### [Stack](http://facebook.github.io/immutable-js/docs/#/Stack) <a name="Stack"></a>
  
  A [stack](http://en.wikipedia.org/wiki/Stack_%28abstract_data_type%29) is a data structure where items are always inserted at the top (or beginning) and removed from the top.
  This data structure is extremely efficient at inserting and removing elements but horribly inefficient at accessing any element in the data structure with the exception of the first one. 

  _When would this be useful?_
  > Because of the way immutable works, something like a state history would be perfect for a stack.
  >
  > First create a stack
  >
  > every time state changes, push it onto the stack
  >
  > Create a button that sets State = stack.pop();
  >
  > Every time the user clicks that button, they will be transported back in time.

--------------------------------------------------------------------------------------------------------------

### [Seq](http://facebook.github.io/immutable-js/docs/#/Seq) <a name="Seq"></a>
  
  An Seq is like a list but its lazy, so operations you apply to them won't be performed until the sequence is accessed. Which has intense implications. 
  I won't say too much on sequences because this is one of the few areas that ImmutableJS is well documented.

--------------------------------------------------------------------------------------------------------------

### [KeyedSeq](http://facebook.github.io/immutable-js/docs/#/KeyedSeq) <a name="KeyedSeq"></a>

  If a Seq were an array, this would be an object. Similar to sequence in that they are lazy, however it works with keys instead of indices.

--------------------------------------------------------------------------------------------------------------

### [SetSeq](http://facebook.github.io/immutable-js/docs/#/SetSeq) <a name="SetSeq"></a>
  
  I must admit I am a big confused by this one. Feel free to fork and submit a pull request with an explanation.
  The reason I am confused is because:
  >Because Seq are often lazy, SetSeq does not provide the same guarantee of value uniqueness as the concrete Set.
  I don't see the purpose of having a set that doesn't guaruntee uniqueness and don't recommend using this unless you have a good reason.




<!-- {% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %} -->