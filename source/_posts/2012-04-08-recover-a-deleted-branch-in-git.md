---
layout: post
title:  Recover a deleted branch in Git
categories:
    - blog
---
In general I delete branches as I am finished with them. Sometimes however, I end up with multiple branches hanging around so I end up needing to clean up. It is on these occasions, that I have accidentally deleted branches I still need.

This isn't a problem in Git however, because we can easily create a new branch and merge the commits from our old deleted branch into it. The process is simple:

```
git checkout -b our-new-branch
git reflog
```

reflog displays a list of changes to the tips of our branches. Something like:

```
---
ebe0eed HEAD@{16}: commit (amend): Initial work on refactoring the extension load mechanism
7d1cb9e HEAD@{19}: rebase -i (finish): returning to refs/heads/feature/module-loader
7d1cb9e HEAD@{20}: rebase -i (pick): Initial work on refactoring the extension load mechanism
610e270 HEAD@{21}: checkout: moving from feature/module-loader to 610e2702df6e5f1d31dcaabeef2a81fc51346d35
6584aca HEAD@{22}: checkout: moving from develop to feature/module-loader
610e270 HEAD@{23}: merge hotfix/fix-reouter-event-names-in-tests: Merge made by the 'recursive' strategy.
025547b HEAD@{24}: checkout: moving from hotfix/fix-reouter-event-names-in-tests to develop
---
```

ebe0eed is the commit I want. Merge it into the current branch:

```
git merge ebe0eed
```

Done.
