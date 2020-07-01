---
layout: post
title: "[out of date, wrong] How to stream HTTPS in chunks with Ruby's Net::HTTP"
description: "So you don't load it all into memory"
category:
tags: [ruby]
---
{% include JB/setup %}

_WARNING: This post is ancient and probably wrong._

Standard Net::HTTP usage pulled from the [Ruby 1.9.3 docs for Net::HTTP](http://ruby-doc.org/stdlib-1.9.3/libdoc/net/http/rdoc/Net.html):

{% highlight ruby %}
require 'tmpdir'
require 'uri'
require 'net/http'

# works with either a query already in the URL or as a second argument
def download_to_tmp_path(url, query = nil)
  uri = URI(url)
  tmp_path = File.join Dir.mktmpdir(uri.to_s.gsub(/\W/, '_')), "out"
  if query
    uri.query = URI.encode_www_form query
  end
  Net::HTTP.start(uri.host, uri.port, :use_ssl => (uri.scheme == 'https')) do |http|
    request = Net::HTTP::Get.new uri.request_uri
    http.request request do |response|
      File.open tmp_path, 'w' do |io|
        response.read_body do |chunk|
          io.write chunk
        end
      end
    end
  end
  tmp_path
end
{% endhighlight %}

so for example

{% highlight ruby %}
puts "HTTP"
url = 'http://s3.amazonaws.com/creative.faraday.io/logo.png'
logo = download_to_tmp_path(url)
system 'file', logo

puts
puts "HTTPS"
url = 'https://s3.amazonaws.com/creative.faraday.io/logo.png'
logo = download_to_tmp_path(url)
system 'file', logo

puts
puts "Inline query"
url = 'https://s3.amazonaws.com/creative.faraday.io/logo.png?greeting=hello+world'
logo = download_to_tmp_path(url)
system 'file', logo

puts
puts "Query as hash"
url = 'https://s3.amazonaws.com/creative.faraday.io/logo.png'
query = { greeting: 'hello world' }
logo = download_to_tmp_path(url, query)
system 'file', logo
{% endhighlight %}

gives

{% highlight console %}
HTTP
http://s3.amazonaws.com/creative.faraday.io/logo.png
/var/folders/cw/h96jw4sj6b19gvk9cnsy15h40000gp/T/http___s3_amazonaws_com_creative_faraday_io_logo_png20140714-4167-miomm3/out: PNG image data, 134 x 40, 8-bit/color RGBA, non-interlaced

HTTPS
https://s3.amazonaws.com/creative.faraday.io/logo.png
/var/folders/cw/h96jw4sj6b19gvk9cnsy15h40000gp/T/https___s3_amazonaws_com_creative_faraday_io_logo_png20140714-4167-ud6zhp/out: PNG image data, 134 x 40, 8-bit/color RGBA, non-interlaced

Inline query
https://s3.amazonaws.com/creative.faraday.io/logo.png?greeting=hello+world
/var/folders/cw/h96jw4sj6b19gvk9cnsy15h40000gp/T/https___s3_amazonaws_com_creative_faraday_io_logo_png_greeting_hello_world20140714-4167-143mn3q/out: PNG image data, 134 x 40, 8-bit/color RGBA, non-interlaced

Query as hash
https://s3.amazonaws.com/creative.faraday.io/logo.png
/var/folders/cw/h96jw4sj6b19gvk9cnsy15h40000gp/T/https___s3_amazonaws_com_creative_faraday_io_logo_png20140714-4167-79adl/out: PNG image data, 134 x 40, 8-bit/color RGBA, non-interlaced
{% endhighlight %}
