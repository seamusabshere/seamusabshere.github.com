---
layout: post
title: "Ruby to_i considered harmful - what to do instead"
description: "You knew to_i was brutal, but this brutal?"
category: 
tags: [ruby]
---
{% include JB/setup %}

I recommend using `to_f.round` instead of `to_i` any day of the week.

{% highlight console %}
>> '1.5'.to_i
=> 1
>> '1.5'.to_f.round
=> 2   # arguably better
{% endhighlight %}

Even stronger evidence:

{% highlight console %}
>> '1.5e3'.to_i
=> 1
>> '1.5e3'.to_f.round
=> 1500 # incontrovertibly better
{% endhighlight %}

I think a lot of Rubyists were introduced to `to_i` from DHH's slug hack involving redefining `Post#to_param` (`"15-hello-world".to_i #=> 15`), but I just don't see many other uses for `to_i`.
