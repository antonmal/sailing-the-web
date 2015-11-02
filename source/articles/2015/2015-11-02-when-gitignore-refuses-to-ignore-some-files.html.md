---
layout: post
title: When .gitignore refuses to ignore some files
date: 2015-11-02 15:38 UTC
tags: git
image: ignoring.jpg
---

![Ignoring](ignoring.jpg)

I've been working on a few Ruby in Rails projects. At some point I realized that some of the files should not be tracked by GIT. Some of them contain sensitive information. Others are temporary or system files, or file that are only relevant to my local machine and setup. It would be a waste to push those to GitHub.

I found a few ```.gitignore``` [templates][gt] (and [more][mgt]) to un-track these types of files. However, when I change my ```.gitignore``` file, I noticed that GIT does not pay attention to these changes and keeps tracking those unwanted files. How can this be?

[gt]: https://www.gitignore.io/
[mgt]: https://github.com/github/gitignore

It turns out that that since GIT tracked these files before I changed the ```.gitignore``` settings, it will keep doing this despite what ```.gitignore``` says.

Here is how to solve this problem:

1. Commit all your recent changes. Otherwise you will loose them.  
*(Yes, the files that you want to be ignored will still be tracked and committed at this stage).*

2. Un-track ALL files in the current folder:

```shell
$ git rm -r --cached .
```

3. Track all the files again:  
*This time git will pay attention to ```.gitignore``` and track only the files you need.*

```shell
$ git add .
$ git commit -m "Remove gitignored files"
```

> Like this solution? Share it with friends!

> You approach this differently? Please, share in the comments!
