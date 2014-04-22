---
layout: post
title: "How to convert a Zip9 or Zip4 to a Zip5 if you're missing leading zeros"
description: "For when Excel turns 057531234 into 57531234.0 and you just want 05753"
category: 
tags: [ruby]
---
{% include JB/setup %}

For when Excel turns `057531234` into `57531234.0` (drops leading zeros and treats it like a number) and you just want `05753`.

Here's the problem in the form of an acceptance test:

{% highlight ruby %}
require 'minitest/autorun'

describe 'zip5 disambiguation' do
  {
    999501234  => '99950',
  '99950-1234' => '99950',
     57531234  => '05753',
   '5753-1234' => '05753',
  '05753-1234' => '05753',
      5011234  => '00501',
    99950      => '99950',
     5753      => '05753',
     1000      => '01000',
      501      => '00501',
  }.each do |input, expected|
    it "parses #{input.inspect} as #{expected.inspect}" do
      zip5(input).must_equal expected
    end
    # here we also test 57531234.0, etc.
    if input.is_a?(Integer)
      input_f = input.to_f
      it "parses #{input_f.inspect} as #{expected.inspect}" do
        zip5(input_f).must_equal expected
      end
    end
  end
end
{% endhighlight %}

Here's a quick Ruby method to solve it:

{% highlight ruby %}
def zip5(input)
  input = input.to_i.to_s.delete('-')
  zip5 = case input.length
  when 9
    input[0,5]
  when 8
    '0' + input[0,4]
  when 7
    '00' + input[0,3]
  when 5
    input
  when 4
    '0' + input
  when 3
    '00' + input
  else
    raise "can't derive zip5 from #{input.inspect}"
  end
  unless (501..99950).include?(zip5.to_i)
    $stderr.puts "warning: looks like a bad zip5 (expected 00500..99950): #{zip5}"
  end
  zip5
end
{% endhighlight %}

Works:

{% highlight ruby %}
Run options: --seed 32751

## Running:

.................

Finished in 0.002172s, 7826.8877 runs/s, 7826.8877 assertions/s.

17 runs, 17 assertions, 0 failures, 0 errors, 0 skips
{% endhighlight %}

You could take the warning out if you want - technically [`501..99950` is full range of zip codes](http://en.wikipedia.org/wiki/ZIP_code#By_geography).
