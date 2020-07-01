---
layout: post
title: "[wrong] How to move a distance in meters away from a point in Ruby using RGeo"
description: "For example, what is 100 meters west of POINT(-72.4861 44.1853)?"
category:
tags: [ruby, gis]
---
{% include JB/setup %}

_WARNING: This post is ancient and wrong._

Here's a method that lets you move/offset/translate a certain distance (in meters) away from a point expressed in longitude and latitude:

{% highlight ruby %}
a = move_point(-72.4861, 44.1853, 0, 0)     # POINT (-72.4861 44.18529999999999)
b = move_point(-72.4861, 44.1853, 100, 0)   # POINT (-72.48520168471588 44.18529999999999)
c = move_point(-72.4861, 44.1853, 0, 100)   # POINT (-72.4861 44.18594416889434)
puts a.distance(b)
puts a.distance(c)
{% endhighlight %}

This gives you:

{% highlight console %}
99.99999999906868
99.99999999906868
{% endhighlight %}

It uses the [`RGeo`](https://github.com/rgeo/rgeo) library. Note the argument order is longitude, latitude!

{% highlight ruby %}
require 'rgeo'

def move_point(lon, lat, x_offset_meters, y_offset_meters)
  wgs84 = RGeo::Geographic.simple_mercator_factory.point(lon, lat)
  wgs84_factory = wgs84.factory
  webmercator = wgs84_factory.project wgs84
  webmercator_factory = webmercator.factory
  webmercator_moved = webmercator_factory.point(webmercator.x+x_offset_meters, webmercator.y+y_offset_meters)
  wgs84_factory.unproject webmercator_moved
end
{% endhighlight %}

