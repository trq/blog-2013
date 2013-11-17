---
layout: post
title:  Git patches
categories:
    - blog
---
Some projects require new developers to submit changes as patch sets before you get write access to there main repositories.

I'm going to show you how to create patches directly from [git][Git]. The process is simple.

Before starting work create a new branch from your up to date master branch, make your changes and commit them. You then simply execute:

```
$ git format-patch master --stdout > ~/mywork.patch
```

This will create a patch containing the differences between your 'master' branch and the new branch that you have just completed your work in and save it within your ~/ directory.

If anyone ever sends you a patch that you need to apply to a [git][Git] repository, [git][Git] makes that easy too.

Once again, this should be done in a new clean branch created from your master branch.

```
$ git apply --stat ~/someones.patch
```

This doesn't actually apply the patch, but will describe what the patch is about to do. It's handy to see if there are going to be any conflicts. All going well you can apply the patch:

```
$ git am --signoff < ~/someones.patch
```

This will apply and commit the patch, creating a signed-off by message in your log.

[git]:      http://git-scm.com
