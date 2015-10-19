---
layout: post
title: How to use long strings in Ruby
date: 2015-10-18 20:11 UTC
tags: ruby
---

I really like to keep my code neat and up to the standards of [Ruby Style Guide][ruby-style] and the [Rubocop][rubocop] code validator. This includes keeping the lines of code limited to 80 characters. But sometimes it's just not possible. What's best to do in this case?

[ruby-style]: https://github.com/bbatsov/ruby-style-guide
[rubocop]: http://batsov.com/rubocop/

For example, I made a hash of possible error messages for my app, some of which were much longer than 80 chars.

~~~ruby
{
  key1: "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod  tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate.",

  keyN: "..."
}
~~~

To make the hash more readable, I wanted to **tab the values neatly to the right from the key**. How would I do that?

### Why not simply tab?

Tabbing the different lines not only added a lot of manual work, but also (surprise!) lead to new-line characters and tabs being added to the string:

~~~ruby
key:  "Lorem ipsum dolor sit amet, consectetur adipisicing
      elit, sed do eiusmod tempor incididunt ut labore et dolore
      magna aliqua. Ut enim ad minim veniam, quis nostrud
      exercitation ullamco laboris nisi ut aliquip ex ea commodo
      consequat. Duis aute irure dolor in reprehenderit in voluptate."

=> "Lorem ipsum dolor sit amet, consectetur adipisicing\n      elit, sed do eiusmod tempor incididunt ut labore et dolore\n      magna aliqua. Ut enim ad minim veniam, quis nostrud\n      exercitation ullamco laboris nisi ut aliquip ex ea commodo\n      consequat. Duis aute irure dolor in reprehenderit in voluptate."
~~~

### Add ```%()```?

At first this looked like a promising solution:

~~~ruby
key:  %(Lorem ipsum dolor sit amet, consectetur adipisicing
      elit, sed do eiusmod tempor incididunt ut labore et dolore
      magna aliqua. Ut enim ad minim veniam, quis nostrud
      exercitation ullamco laboris nisi ut aliquip ex ea commodo
      consequat. Duis aute irure dolor in reprehenderit in voluptate.)

=> "Lorem ipsum dolor sit amet, consectetur adipisicing\n      elit, sed do eiusmod tempor incididunt ut labore et dolore\n      magna aliqua. Ut enim ad minim veniam, quis nostrud\n      exercitation ullamco laboris nisi ut aliquip ex ea commodo\n      consequat. Duis aute irure dolor in reprehenderit in voluptate."
~~~

...but turned out to have exactly the same issues.

### Add ```\``` at the end of each line?

The next trick helped me get rid of the new lines:

~~~ruby
key:  %(Lorem ipsum dolor sit amet, consectetur adipisicing \
      elit, sed do eiusmod tempor incididunt ut labore et dolore \
      magna aliqua. Ut enim ad minim veniam, quis nostrud \
      exercitation ullamco laboris nisi ut aliquip ex ea commodo \
      consequat. Duis aute irure dolor in reprehenderit in voluptate.)

=> "Lorem ipsum dolor sit amet, consectetur adipisicing       elit, sed do eiusmod tempor incididunt ut labore et dolore       magna aliqua. Ut enim ad minim veniam, quis nostrud       exercitation ullamco laboris nisi ut aliquip ex ea commodo       consequat. Duis aute irure dolor in reprehenderit in voluptate."
~~~

...but tabs still persisted. And this approach meant even more manual work for me.

### Ask ```%w()``` for help?

The next, slightly unconventional approach actually worked (kind of). Notice the ```%w``` construct that breaks the string into array of words and the ``` * ' '``` array operator that joins it back together (equivalent to ```.join(' ')```):

~~~ruby
key:  %w(Lorem ipsum dolor sit amet, consectetur adipisicing
      elit, sed do eiusmod tempor incididunt       ut labore et dolore
      magna aliqua. \nUt enim ad minim veniam, quis nostrud
      exercitation ullamco laboris nisi ut aliquip ex ea commodo
      consequat. Duis aute irure dolor in reprehenderit in voluptate.) * ' '

=> "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. \\nUt enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate."
~~~

Then I realized that this approach would not work if I needed some ```\n``` or similar characters or more than one space between some words in the string, as shown in the above example. Back to square one.

### Use heredocs?

I tried using so-called ['heredocs'][heredoc] which look like this:

[heredoc]: https://en.wikibooks.org/wiki/Ruby_Programming/Here_documents

~~~ruby
key:  <<-EOS
        Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
        magna aliqua. \nUt enim ad minim veniam, quis nostrud exercitation
        ullamco laboris nisi ut aliquip ex ea      commodo consequat. Duis aute
        irure dolor in reprehenderit in voluptate.
      EOS

=> "        Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod\n        magna aliqua. \nUt enim ad minim veniam, quis nostrud exercitation\n        ullamco laboris nisi ut aliquip ex ea      commodo consequat. Duis aute\n        irure dolor in reprehenderit in voluptate.\n"
~~~

The extra new lines and tabs are back again. Plus some extra space in the front and another new line in the end. And some ugly not-quite-Ruby syntax.

### Write a method?

I found a solution to somewhat similar problem in Rails, which involved creating a custom method to remove the extra indentation from inside the string. I tried to apply the same general approach and wrote a Ruby (non-Rails) method:

~~~ruby
class String
  def undent
    self.split(/^[ \t]+/).map(&:rstrip).join
  end
end

key:  "Lorem ipsum dolor sit amet, consectetur          adipisicing
      elit, sed do eiusmod tempor incididunt ut labore et dolore
      magna aliqua. \nUt enim ad minim veniam, quis nostrud
      exercitation ullamco laboris nisi ut aliquip ex ea commodo
      consequat. Duis aute irure dolor in reprehenderit in
      voluptate.".undent

=> "Lorem ipsum dolor sit amet, consectetur          adipisicingelit, sed do eiusmod tempor incididunt ut labore et doloremagna aliqua. \nUt enim ad minim veniam, quis nostrudexercitation ullamco laboris nisi ut aliquip ex ea commodoconsequat. Duis aute irure dolor in reprehenderit involuptate."
~~~

Now, this worked quite nicely and did not mind the extra spacing and ```\n``` characters. Putting this method in some kind of utilities library could be a good solution, but may be an overkill for smaller projects.

### Use an array of lines?

One other option:

~~~ruby
key:  ['Lorem ipsum dolor sit amet, consectetur adipisicing',
      'elit, sed do eiusmod tempor incididunt       ut labore et dolore',
      'magna aliqua. \nUt enim ad minim veniam, quis nostrud',
      'exercitation ullamco laboris nisi ut aliquip ex ea commodo',
      'consequat. Duis aute irure dolor in reprehenderit in voluptate.']
      .join

=> "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. \\nUt enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate."
~~~

This is a workable solution. Quite a lot of work putting each line in quotes, though. There is a shortcut for that. In Sublime or Atom you can work on several lines at once. I use (and love) Atom and there you can select all lines and press 'Shift-Cmd-L' to enclose each of them in ```'``` and then add ```,``` at the end.

### Plain old string concatenation?

If you need to add quotes to each line, why use arrays at all? Plainly joining parts of the string together with ```+``` or ```<<``` will work just as well and will be more readable. Do not forget to use the shortcut from the previous option.

~~~ruby
key:  'Lorem ipsum dolor sit amet, consectetur adipisicing' +
      'elit, sed do eiusmod tempor incididunt       ut labore et dolore' +
      'magna aliqua. \nUt enim ad minim veniam, quis nostrud' +
      'exercitation ullamco laboris nisi ut aliquip ex ea commodo' +
      'consequat. Duis aute irure dolor in reprehenderit in voluptate.'

=> "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. \\nUt enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate."
~~~

### Use modern editors to avoid the problem altogether?

If you can be sure that your team only uses good code editors like Sublime or Atom, they can be tweaked to support the needed auto-indentation effect without breaking the text into several lines and tabbing them. You can achieve that by turning word wrap on and setting the threshold to 80 characters. Sublime does a good job of this:

![Sublime auto-indentation](sublime-auto-indent.png)

...and Atom works even better:

![Atom auto-indentation](atom-auto-indent.png)

## CONCLUSION

Unexpectedly, this post ran very long. You may say that I went into too much trouble to solve a tiny, even imaginary problem. But, I do hope that some of these approaches were new to you and you will use them one day to solve a real problem.

It would be nice for Ruby to have a special construct similar to ```%()```, for example ```%untab()``` or ```%u()``` for short, to tell it that the following tabbed lines should be concatenated together stripping the extra indentation. Without such construct, I feel like none of the above solutions are worth using in light of having the code editors capable of accomplishing the same effect automatically.

> What approach would you use? Did I miss any?
