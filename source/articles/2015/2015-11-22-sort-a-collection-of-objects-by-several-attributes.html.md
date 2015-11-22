---
layout: post
title: "Sort a collection of objects by several attributes"
date: 2015-11-22 19:25 UTC
tags: ['rails', 'tealeaf']
image: sorting.jpg
---

![Sorting](sorting.jpg)

While working on the [PostIt app][pi] as part of the [Tealeaf course][tl], I needed to sort a collection of posts by two parameters: the sum of all votes for/against the post and the date when the post was last updated.

[tl]: https://www.gotealeaf.com
[pi]: https://tl-postit-amalkov.herokuapp.com

After struggling with monstrous constructs, custom SQL requests (not too welcome in Rails) and the like, I came up with a pretty elegant way to do this:

```ruby
# posts_controller.rb
@posts = Post.all.sort_by { |post| [-post.votes_count, -post.updated_at.to_i] }

# my 'voteable_ant' gem
def votes_count
  up_votes - down_votes
end

def up_votes
  self.votes.where(vote: true).size
end

def down_votes
  self.votes.where(vote: false).size
end
```

Here sorting happens by building a two-element array for each post and comparing these arrays to each other. The way such comparison (```<=>```) happens in Ruby is that initially, the first elements of those arrays are compared and, if those are equal, then the second ones are compared between each other. So, in the end, the ```Enumerable``` containing a collection of ```Post``` objects gets sorted by ```.votes_count``` first and by ```.updated_at``` second.

The ```-``` sign acts just like ```DESC``` in SQL, it reverses the sort order of a particular array element from ascending to descending. Since ```.updated_at``` returns a DateTime object, which cannot be negated (at least in Ruby), I applied ```.to_i``` to turn it into a timestamp (essentially an integer). After this negation worked just fine.

What if one of the parameters was a ```String```? Not as elegant, but still reasonable solution could look like this:

```ruby
@posts = Post.all.sort_by { |post| [
  post.some_string.chars.map{|e| -e.ord},
  post.updated_at.to_i
  ] }
```

Here I turn the string into an array of ASCII (again, basically integer) values representing each character and negate the ```-e.ord``` to make those integers negative and thus sort the strings in a descending order.

> How would you approach this?
