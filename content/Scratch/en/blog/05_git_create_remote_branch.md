-----
kind: article
theme: brutalist
menupriority: 1
published: 2009-08-17
title: Git remote branch creation
author: Yann Esposito
authoruri: yannesposito.com
tags:  git, branch, local, remote
-----

## easiest remote Git branch creation

I use git simply to synchronize stuff for personnal projects.
Therefore, when I create a local branch I want most of time this
branch to be created remotely.

Here is the script I use to achieve that: 

~~~zsh
#!/usr/bin/env zsh

if (($#<1)); then
    print -- "usage: $0:t branch_name" >&2
    exit 1
fi

branch=$1
git br ${branch}
git co ${branch}
git config branch.${branch}.remote origin
git config branch.${branch}.merge refs/heads/${branch}
~~~

Of course, I suppose `origin` is already configured.

*Edit: Now I use `git push -u`*
