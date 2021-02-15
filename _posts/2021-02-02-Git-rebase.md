---
layout: post
title: Git rebase explained in 3 steps
thumbnail-img: /assets/img/metro.jpg
tags: [git, rebase]
---

It's sometimes amazing how Git facilitates coder's life. The `rebase` operation is one of my favorite because I used it regularly, even when I work alone on a project. To those who wonders what that does and how to use it, I wrote this little article.

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
