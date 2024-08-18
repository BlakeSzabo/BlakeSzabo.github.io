---
title: Big O is Lying to You 
date: 2024-08-17 21:00:00+1000
categories: [Computer Science]
tags: [Computer Science]
author: blake_szabo
description: What they won't tell you in DSA Wall St. 
---

Tell me, what's faster? An `O(log n)` time complexity algorithm or an `O(n^2)` time complexity algorithm? The answer is simple: 
I don't know; and neither do you. I often see various algorithms recommended because
the time complexity is _smaller_. I mean, clearly `log(n)` is smaller than `n^2`, right?
In fact, check out this graph:

![Graph plot of x^2 and ln(x)](/assets/img/pages/Big-O-is-Lying-to-You/one.png)
_Graph plot of x^2 and ln(x)_

The polls are in, `log(n)` is better than `n^2` all the time, it even goes into negative time when n is smaller than one.
You can't beat time travel, buddy. Except, there's a few crucial
details missing here, allow me to explain.

# What is Big O?

The first thing we need to tackle is: What even is Big O? To put it simply, and thus opening myself
to the anonymous commenter "well actually" attack vector, Big O is a notation to denote an algorithm's rate of growth
in time or space (memory usage, not astral space) relative to zero or more inputs. It is represented
with a literal big O, chosen to mean "order of approximation".

That means that when we say an algorithm has a time complexity of `O(n)`, it means that we expect
the execution time to increase _approximately_ linearly with respect to the input size, n.
The key word here? Approximately. Big O notation, in my humble opinion, is meant to serve
as a quick reference for how you might expect your algorithm to perform at scale. To that
end, the complexity of an algorithm is only determined by the largest influencing factor.
For example, if your time complexity is `n + log(n)` then think again, it's actually just
`n`. Why? You have to remember, this is **BIG** O, not small o and not medium 0. The measurement
is at large scale, which at that point the `log n` portion of your execution becomes basically
a rounding error. For example, if your algorithm is `n + log(n)` and n is 10,000,000: That means
`n + log(n)` will be equal to `10,000,000 + ~16`<sup>*</sup>.
<br /><sup><sub>* assuming log is just natural log</sub></sup>

We've established all we need to know to illustrate the problem with the original question.
You see, we also trim constant values that aren't impacted by the input size. Unless the algorithm's
complexity is only constant in which we say it is O(1). So what if our `log(n)` algorithm actually
has a secret initialization step that makes it something like `O(log n + 15)`? When we're working
with really small input, the `O(n^2)` algorithm might actually be "faster" than the `O(log n)` one.

Now, in reality at small scale this doesn't even matter. If you're thinking whether you should use
QuickSort or Insertion Sort on an array of 4 items, you'll probably waste more time reading this sentence
than you'd ever save by using whatever ends up being quickest. Computers are _fast_, that's why we have
Big O notation in the first place; the execution time for computers tends to become tangible at larger
input sizes. Much larger.

# What, exactly, are you doing?

Another thing to consider is what you're going to be doing. Now, that does include how big
the inputs you'll be working with are, but it also includes some other considerations. Take
for example a simple linear search vs binary search on an array. Binary search has a small
catch with it in that the array you're searching has to be sorted. Is the cost of sorting
worth the benefit of a binary search? Well if you're just doing _one_ search then it probably
isn't. However, things don't always happen in isolation. If you sort that array once but then
do multiple searches on the array, you'll make your money back quite quickly. You'll find these
tradeoffs quite often, where specific algorithms or data structures are specialized for specific
tasks. Who knew?

I implore you, think about your use case. Many times I have worried about the time complexity
of some algorithm I'm implementing thinking, "How will this impact the economy?", without
taking a step back and realizing; I'm sorting, at most, 100 things. Who cares? I'm willing to bet
that many a time this will also be you. You should really be thinking about Big O when you're 
dealing with real scale. Not human scale, computer scale. Until then, it's better to
pick a reasonable, simple solution that gets the job done. If I could pick any one problem
to have for the rest of my life, it would be scale problems. At least then, I know I am
succeeding.

# The Graphs also LIE

Another small thing I wanted to touch on is the fact that the graphs you usually see
with a nice smooth curve showing the execution time based on the input are kind of a lie.
That's because execution time is pretty much a direct consequence of the number of instructions
you execute, which is a discrete measurement. My favourite go-to example algorithm, as I'm sure
you've noticed, binary search shows this off quite well. Every iteration binary search effectively
cuts the search space in half. In the worst case, you'll divide in 2 until there's
one item left, which will either be your search item or your search item isn't in the list.
To keep things simple I'm going to assert that we can represent the number of comparisons
here with a base 2 log of n, where n is the size of the input array.

![Graph plot of base 2 log of n](/assets/img/pages/Big-O-is-Lying-to-You/two.png)

Because there's only a descrete number of comparisons though, it will actually look
something more like this:

![Graph plot of base 2 log of n, where the output is rounded up](/assets/img/pages/Big-O-is-Lying-to-You/three.png)

This isn't really as major of an impact on performance so much as it is just another
lie fabricated by Big O to make you _think_ you know what's going on.

# Why should I care?

Well, if your work day is anything like mine then you probably won't need to use
this information very often. I still think it is good though, to know about what
we actually mean when we talk about time/space complexity. It is not an absolute
measure of this algorithm is better, faster or more memory efficient than another.
It _is_ a measurement of how an algorithm will scale with respect to the input.
The moral of the story is that with anything performance driven, it's complicated.
Ideally, you should practically test the options in your solution space and land
on what you think is reasonable. Unless you have a good reason to I generally don't
think it would be a good use of your time to spend months implementing a complicated
solution to a problem you have to save 0.5ms before you actually ship something that works.

Stay curious.
