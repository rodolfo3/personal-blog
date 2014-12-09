---
layout: post
title:  "Jekyll (without sudo)!"
date:   2014-12-11 00:00:00
categories: blog debian
---

Debian has his own package manager (apt-get). Ruby has one too (gem).

Trying to install jekyll on Debian using gems (since it's not available using apt-get) needs root access (or sudo) and can overwrite some other package files (while installing his dependencies).

A simple solution is use environment vars and install it in $HOME directory.

{% highlight bash %}
$ export GEM_HOME="$HOME/.gems"
$ gem install jekyll
{% endhighlight %}

If you get some errors, you may are using other jekyll (from outsige ".gems"). Set your `$PATH` too:

{% highlight bash %}
$ export PATH="$GEM_HOME/bin:$PATH"
{% endhighlight %}

And that's it!

{% highlight bash %}
$ jekyll new .
$ jekyll serve --watch
{% endhighlight %}
