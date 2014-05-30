---
layout: post
title:  "Lets talk about Ruby"
date:   2014-05-30 00:30:22
categories: ruby
author: Douglas Rossignolli
---

For our first publication we will talk about ruby, there so many people starting with ruby these days, the community is growing up __everyday__, here in Brazil (where i live), i've seen the number of companies using ruby growing with the demand of ruby developers.

So we'll try to help the people who wants to learn about ruby and your gems.

Lets get our hello world from ruby here today.

For the first, I really loves [RVM] (Ruby Version Manager), many developers will say [rbenv] is better, but here you will use RVM, but you are free to use others managers if you want, or if you would like to use the ruby without any manager, it's fine too.

Lets install RVM
{% highlight bash %}
	$ \curl -sSL https://get.rvm.io | bash -s stable
{% endhighlight %}

Note: if you are using mac you will perhaps need to install wget and brew, 
`ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"`,
also if you are using ubuntu you can use `sudo apt-get install wget`

Then we can install our requirements using the command
{% highlight bash %}
	$ rvm requirements
{% endhighlight %}

Note: try to run `rvm -h` to get help about the functions of RVM, one thing to do after instalations is run
`rvm requirements` to installs additional OS specific dependencies/requirements for building various rubies. Usually run by install.

Now we'll install the last version of ruby
{% highlight bash %}
	$ rvm install 2.1.0
{% endhighlight %}

Tip: if you want to know what are the available versions use the command `rvm list known`

Okay! Lets go to Hello World
Our first class on Ruby, lets create our file using vi/vim

`vim hello.rb`

{% highlight ruby %}
# hello.rb
class Hello
  def initialiser
    puts "This is our class Hello"
  end

  def say_hello_to person
    puts "Hello #{person}"
  end
end
{% endhighlight %}

Now we need to use our first class `vim test.rb`

{% highlight ruby %}
# test.rb
require 'hello'

hello = Hello.new
hello.say_hello_to "Douglas"

{% endhighlight %}

Now just run our `test.rb` file just like this `$ ruby test.rb`
The result is `Hello Douglas`

I hope you enjoy this introduction to the ruby!

[RVM]: http://rvm.io/
[rbenv]: https://github.com/sstephenson/rbenv
[brew]: http://brew.sh/