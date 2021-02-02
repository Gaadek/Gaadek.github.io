---
layout: post
title: Git rebase explained to my little sister
tags: [git, rebase]
---

It's sometimes amazing how afraid are some people when they hear `git`. And honestly, it was (is?) my case, I'm far from mastering git, so I need small sticky notes to remember ho do do things well. Today I'll write to myself how to perform a git rebase and what it does.

## Scenario
1) A branch `new-feature` has been created from branch `master`
```
A---B ← master
     \
      C ← new-feature
```

2) **After** `new-feature` has been created, another branch `bug-fix` has been created from `master` then merged into
```
      D ← bug-fix
     / \
A---B---E ← master
     \
      C ← new-feature
```


3) You'd like to get the updates from `bug-fix` into `new-feature`
```
      D ← bug-fix
     / \
A---B---E ← master
         \
          C ← new-feature
```

## Process
**Step #1:** Get the latest master branch which contains the `bug-fix` commits  
```shell
$ git checkout master
$ git pull
```

**Step #2:** Checkout the `new-feature` branch
```shell
$ git checkout new-feature
```

**Step #3:** Rebase `new-feature` branch with master
```shell
$ git rebase master
```

**Step #4:** that's it, you just did it
