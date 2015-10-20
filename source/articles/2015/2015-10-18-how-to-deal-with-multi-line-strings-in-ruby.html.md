---
layout: post
title: How to use long strings in Ruby
date: 2015-10-18 20:11 UTC
tags: ruby
image: 'long-bridge.jpg'
---

![Endless bridge](long-bridge.jpg)

I really like to keep my code neat and up to the standards of [Ruby Style Guide][ruby-style] and the [Rubocop][rubocop] code validator suggestions. This includes keeping the lines of code limited to 80 characters. But sometimes they do not fit. What is best to do in such case?

[ruby-style]: https://github.com/bbatsov/ruby-style-guide/
[rubocop]: http://batsov.com/rubocop/

For example, I made a structured hash of messages for my app, some of which were much longer than 80 chars.

~~~ruby
{
  key: {
    subkey: {
      subsubkey: "Lorem ipsum dolor sit amet,     consectetur adipisicing elit, sed do eiusmod magna aliqua. \nUt enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate."
    }
  }
}
~~~

Scroll the code block to the right. As you can see, the long string went way beyond the limit of the code block, just as it would in GitHub code viewer and many other places. How can I make that string more easily readable?

### Why not simply break the string into several lines?

~~~ruby
{
  key: {
    subkey: {
      subsubkey: "Lorem ipsum dolor sit amet,     consectetur
adipisicing elit, sed do eiusmod magna aliqua. \nUt enim
ad minim veniam, quis nostrud exercitation ullamco laboris
nisi ut aliquip ex ea commodo consequat. Duis aute irure
dolor in reprehenderit in voluptate."
    }
  }
}
~~~

Oops, now all the nicely tabbed hash structure is broken. How can I preserve it?

### Maybe just add [Tab]'s?

~~~ruby
{
  key: {
    subkey: {
      subsubkey: "Lorem ipsum dolor sit amet,     consectetur
      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim
      ad minim veniam, quis nostrud exercitation ullamco laboris
      nisi ut aliquip ex ea commodo consequat. Duis aute irure
      dolor in reprehenderit in voluptate."
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet,     consectetur\n      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim\n      ad minim veniam, quis nostrud exercitation ullamco laboris\n      nisi ut aliquip ex ea commodo consequat. Duis aute irure\n      dolor in reprehenderit in voluptate."```

The code looks nice, as expected. But tabbing each line not only means extra manual work, but also (surprise!) leads to unneeded new-line and tab characters being added to the string. How can I avoid this?

### Use ```%()```?

At first this looked promising:

~~~ruby
{
  key: {
    subkey: {
      subsubkey: %(Lorem ipsum dolor sit amet,     consectetur
      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim
      ad minim veniam, quis nostrud exercitation ullamco laboris
      nisi ut aliquip ex ea commodo consequat. Duis aute irure
      dolor in reprehenderit in voluptate.)
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet,     consectetur\n      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim\n      ad minim veniam, quis nostrud exercitation ullamco laboris\n      nisi ut aliquip ex ea commodo consequat. Duis aute irure\n      dolor in reprehenderit in voluptate."```

...but turned out to have exactly the same issues.

### Add ```\``` at the end of each line?

The next trick helped me get rid of the new lines:

~~~ruby
{
  key: {
    subkey: {
      subsubkey: %(Lorem ipsum dolor sit amet,     consectetur \
      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim \
      ad minim veniam, quis nostrud exercitation ullamco laboris \
      nisi ut aliquip ex ea commodo consequat. Duis aute irure \
      dolor in reprehenderit in voluptate.)
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet,     consectetur       adipisicing elit, sed do eiusmod magna aliqua. \nUt enim       ad minim veniam, quis nostrud exercitation ullamco laboris       nisi ut aliquip ex ea commodo consequat. Duis aute irure       dolor in reprehenderit in voluptate."```

...but tabs still persisted. And this approach meant even more manual work for me.

### Ask ```%w()``` for help?

The next, slightly unconventional approach actually worked (kind of). Notice the ```%w``` construct that breaks the string into array of words and the ``` * ' '``` array operator that joins it back together (equivalent to ```.join(' ')```):

~~~ruby
{
  key: {
    subkey: {
      subsubkey: %w(Lorem ipsum dolor sit amet,     consectetur
      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim
      ad minim veniam, quis nostrud exercitation ullamco laboris
      nisi ut aliquip ex ea commodo consequat. Duis aute irure
      dolor in reprehenderit in voluptate.) * ' '
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod magna aliqua. \\nUt enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate."```

As you can see, this approach works unless the string contains escape characters like ```\n``` or more than one space between some words. Not good. What else can I do?

### Use heredocs?

I tried using so-called ['heredocs'][heredocs], which are basically named text documents incorporated straight into the code with special markup (```<<-NAME ... NAME```):

[heredocs]: https://en.wikibooks.org/wiki/Ruby_Programming/Here_documents/

~~~ruby
{
  key: {
    subkey: {
      subsubkey: <<-STRING
        Lorem ipsum dolor sit amet,     consectetur
        adipisicing elit, sed do eiusmod magna aliqua. \nUt enim
        ad minim veniam, quis nostrud exercitation ullamco laboris
        nisi ut aliquip ex ea commodo consequat. Duis aute irure
        dolor in reprehenderit in voluptate.
      STRING
    }
  }
}
~~~

Value: ```"        Lorem ipsum dolor sit amet,     consectetur\n        adipisicing elit, sed do eiusmod magna aliqua. \nUt enim\n        ad minim veniam, quis nostrud exercitation ullamco laboris\n        nisi ut aliquip ex ea commodo consequat. Duis aute irure\n        dolor in reprehenderit in voluptate.\n"```

The extra new lines and tabs are back again. Plus some extra space at the front and another new line at the end. And some ugly not-quite-Ruby syntax. What's next?

### Write a method?

I found a [solution][strip-heredoc] to somewhat similar problem in Rails, which involved creating a custom method to remove the extra indentation from inside the string. I tried to apply the same general approach and wrote my own Ruby (non-Rails) method:

[strip-heredoc]: http://apidock.com/rails/String/strip_heredoc

~~~ruby
class String
  def undent
    self.split(/^[ \t]+/).map(&:rstrip).join
  end
end

{
  key: {
    subkey: {
      subsubkey: "Lorem ipsum dolor sit amet,     consectetur
      adipisicing elit, sed do eiusmod magna aliqua. \nUt enim
      ad minim veniam, quis nostrud exercitation ullamco laboris
      nisi ut aliquip ex ea commodo consequat. Duis aute irure
      dolor in reprehenderit in voluptate.".undent
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet,     consecteturadipisicing elit, sed do eiusmod magna aliqua. \nUt enimad minim veniam, quis nostrud exercitation ullamco laborisnisi ut aliquip ex ea commodo consequat. Duis aute iruredolor in reprehenderit in voluptate."```

Now, this worked quite nicely and did not mind the extra spacing and escape characters. If you keep some kind of utilities library in your project, put the method there and use it whenever needed. However, this may be an overkill for smaller projects.

### Use an array of lines?

One other option:

~~~ruby
{
  key: {
    subkey: {
      subsubkey: [
        "Lorem ipsum dolor sit amet,     consectetur" ,
        "adipisicing elit, sed do eiusmod magna aliqua. \nUt enim",
        "ad minim veniam, quis nostrud exercitation ullamco laboris",
        "nisi ut aliquip ex ea commodo consequat. Duis aute irure",
        "dolor in reprehenderit in voluptate."
      ].join
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet,     consecteturadipisicing elit, sed do eiusmod magna aliqua. \nUt enimad minim veniam, quis nostrud exercitation ullamco laborisnisi ut aliquip ex ea commodo consequat. Duis aute iruredolor in reprehenderit in voluptate."```

This is a workable solution. Quite a lot of hassle putting each line in quotes. But, there is a shortcut for that. In Sublime or Atom you can work on several lines at once. I use (and love) [Atom][atom] and there you can selecta and unindent all lines, press 'Shift-Cmd-L', then enclose each of them in ```'``` and add ```,```'s at the ends, then indent the lines back.

[atom]: https://atom.io/

### Plain old string concatenation?

If you need to add quotes to each line, why use arrays at all? Plainly joining parts of the string together with ```+``` or ```<<``` will work just as well and will be more readable. (Do not forget to use the shortcut from the previous option.)

~~~ruby
{
  key: {
    subkey: {
      subsubkey:
        "Lorem ipsum dolor sit amet,     consectetur" +
        "adipisicing elit, sed do eiusmod magna aliqua. \nUt enim" +
        "ad minim veniam, quis nostrud exercitation ullamco laboris" +
        "nisi ut aliquip ex ea commodo consequat. Duis aute irure" +
        "dolor in reprehenderit in voluptate."
    }
  }
}
~~~

Value: ```"Lorem ipsum dolor sit amet,     consecteturadipisicing elit, sed do eiusmod magna aliqua. \nUt enimad minim veniam, quis nostrud exercitation ullamco laborisnisi ut aliquip ex ea commodo consequat. Duis aute iruredolor in reprehenderit in voluptate."```

That actually looks fine. But again, a lot of manual work if you have many of those text blocks or if you have to change some of them regularly and thus, put them back together and then re-split into different lines again.

### Use modern editors to avoid the problem altogether?

If you can be sure that your team only uses good code editors like [Sublime][subl] or [Atom][atom], they can be tweaked to support the needed auto-indentation effect without you breaking the text into several lines and tabbing them. You can achieve that by turning word wrap on and setting the threshold to 80 characters. [Sublime][subl] does a good job of this:

[subl]: http://www.sublimetext.com/

![Sublime auto-indentation](sublime-auto-indent.png)

...and [Atom][atom] works even better, I think:

![Atom auto-indentation](atom-auto-indent.png)
___
## CONCLUSION

Unexpectedly, this post ran very long. You may say that I went into too much trouble to solve a tiny, even imaginary problem. But, I do hope that one or two approaches or syntax options were new to you and that you will use those in other ways with much greater gain.

It would be nice for Ruby to have a special construct similar to ```%()```, for example ```%untab()``` (or ```%u()``` for short), to tell it that the following tabbed lines should be concatenated together stripping the extra indentation. Without such construct, none of the above solutions are perfect, but the last 4 of them are quite usable.
___
> What approach do / would you use?

> Did I miss any?

> Share your thoughts in the comments.
