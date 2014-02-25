---
layout: post
title: "Array#include? is slow; use Set#include?"
description: "Don't settle for O(n) when you have O(log n)"
category: 
tags: [ruby]
---
{% include JB/setup %}

Don't use [`Array#include?`](http://ruby-doc.org/core-2.1.1/Array.html#method-i-include-3F) as a lookup table. [`Set#include?`](http://ruby-doc.org/stdlib-2.1.1/libdoc/set/rdoc/Set.html#method-i-include-3F) is way faster.

Here's an example where it's 150X faster:

{% highlight ruby %}
require 'benchmark/ips'
require 'set'

ary = []
5000.times { ary << rand.round(6) }
set = ary.to_set

Benchmark.ips do |x|
  x.report("Array#include?") { ary.include?(rand.round(6))}
  x.report("Set#include?")   { set.include?(rand.round(6))}
end
{% endhighlight %}

And the benchmarking results:

{% highlight console %}
Calculating -------------------------------------
      Array#include?       374 i/100ms
        Set#include?     60669 i/100ms
-------------------------------------------------
      Array#include?     3780.7 (±1.1%) i/s -      19074 in   5.045665s
        Set#include?  2078044.8 (±3.5%) i/s -   10374399 in   5.000050s
{% endhighlight %}
