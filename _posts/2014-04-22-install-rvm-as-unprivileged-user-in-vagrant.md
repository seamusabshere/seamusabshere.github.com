---
layout: post
title: "How to install RVM as an unprivileged user in Vagrant"
description: "If you don't want to install RVM as root in Vagrant, you have to modify the scripts provided on http://rvm.io/integration/vagrant"
category: 
tags: [ruby]
---
{% include JB/setup %}

The [RVM installation instructions for Vagrant](http://rvm.io/integration/vagrant) install [RVM](http://rvm.io) as root.

If you want to install it as an unprivileged user, namely `vagrant`, do this:

{% highlight ruby %}
# http://rvm.io/integration/vagrant - edited to install for "vagrant" user only
config.vm.provision :shell, path: '/host/path/to/install-rvm.sh',  args: 'stable', privileged: false # changed
config.vm.provision :shell, path: '/host/path/to/install-ruby.sh', args: '2.1.1',  privileged: false # changed
{% endhighlight %}

You can use an unmodified `install-rvm.sh`:

{% highlight bash %}
#!/usr/bin/env bash

curl -sSL https://get.rvm.io | bash -s $1
{% endhighlight %}

You should modify `install-ruby.sh` to be:

{% highlight bash %}
#!/usr/bin/env bash

source /home/vagrant/.rvm/scripts/rvm # changed, used to be /usr/local/rvm/scripts/rvm

rvm use --default --install $1        # changed, used to not set --default

shift

if (( $# ))
  then gem install $@
fi
{% endhighlight %}

That's it!
