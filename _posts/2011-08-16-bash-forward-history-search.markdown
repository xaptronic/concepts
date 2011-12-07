---
layout: post
title: Bash forward history search / XON/XOFF
---
History search in bash is a really great feature when you are re running commands while performing some sort of task. For some time I was wondering why the forward history search (ctrl + s) did not work. Eventually I discovered that Ctrl + s is used for flow control.

For example if you type
{% highlight ruby %}
ls -lR
{% endhighlight %}

from the root for example, the terminal will start to list every file on the system. With flow control on you can use Ctrl + s and Ctrl + q to start and stop this flow of output, which I can see having a use if I was running a compilation and wanted to look at some output.

However it is also nice to be able to reclaim Ctrl + s for forward history search. Fortunately, this is very easy:

{% highlight ruby %}
stty -ixon
{% endhighlight %}

disables it and
{% highlight ruby %}
stty ixon
{% endhighlight %}

enables it.

So my default is now to disable it and only enable it when I want to use it for something because I far more often am using history search. I have done this by adding

{% highlight ruby %}
stty -ixon
{% endhighlight %}

to my ~/.bashrc file.
