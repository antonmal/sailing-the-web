---
layout: post
title: "Create GitHub repos from your command line"
date: 2015-10-14 12:03:08 +0200
comments: true
categories: git
---

Repetition is the mother of skill. When you learn web development, you need to expose yourself to the same tasks over and over again. This is how the skills get ingrained in your muscle memory. This includes working on a multitude of different projects one after another to develop your problem solving skills and simply train yourself to create projects from scratch.

Often this means you need to create a new GitHub repo every couple of days, sometimes even hours. After going to GitHub website many times to create a new repo, then copy the link, then go back to the terminal, etc, it started to feel like too much hassle. Being as lazy as I am, I wanted to create these little repos without leaving the command line. I tried to find a git command to do this, but obviously GitHub is not really a part of git, so no such command exists. Then I realized that you can use the command line to send requests to the GitHub API, and the rest fell into place. Here is how you can do this:

1. Get your personal API token from GitHub

    Go to [GitHub personal token page][token] and generate a new API token, if you do not have one already. Default scope settings should work perfectly well for your needs.

[token]: https://github.com/settings/tokens

    !!! Put down or copy the token (the long line of gibberish that will get generated). You only get one chance to do this and will never be shown the token agan.

    ![GitHub tokens](/images/posts/github-token.png)

2. Create a local folder and git repo for the new project

    ~~~shell
    $ mkdir <new-project-name>
    $ cd <new-project-name>
    $ git init
    ~~~

3. Now, the magic: use the token to create new repo through GitHub API

    ~~~shell
    $ curl -u <git-username>:<git-api-token> \
     https://api.github.com/user/repos -d '{"name":"<new-repo-name>"}'
    ~~~

4. Now link the new remote repo with the local one

    ~~~shell
    $ git remote add origin git@github.com:<username>/<new-repo>.git
    $ git push origin -u master
    ~~~

5. Enjoy! ))

*\* I briefly mentioned this trick in the previous post, but then though it deserves it's own.*
