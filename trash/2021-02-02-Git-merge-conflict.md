---
layout: post
title: Handle git merge conflicts with ease
thumbnail-img: /assets/img/metro_complex.jpg
tags: [git, rebase, merge, conflict, meld]
---

In the first episode of this series dedicated to git, I showed how to use `git rebase`. Unfortunatelly, it happens that changes made in the different branches alter the same piece of code.
In such case, git cannot decide alone which change is the most appropriate and will so display in your log there are merge issues:
```shell
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

## Identify files with merge issues
During the `rebase` operation, if a conflict happens, it will be highlighted in the log:
```shell
CONFLICT (content): Merge conflict in path/to/file.go
```

But in case of you're lost, you can at any time get the list of conflicts with a `git status` command:
```shell
$ git status
> # On branch 'new-feature'
> # You have unmerged paths.
> #   (fix conflicts and run "git commit")
> #
> # Unmerged paths:
> #   (use "git add ..." to mark resolution)
> #
> # both modified:      path/to/file.go
```

## Identify merge issues
There, we just have to open files under the **Unmerged path** section (or thos marked **CONFLICT**) with your favorite editor (I personnaly love VSCodium)
Then look for conflicts markers: `<<<<<<<` (incoming change), `=======`, `>>>>>>>` (current change) which appear in the file:
```go
<<<<<<< HEAD
updateEntity(...)
=======
changeEntity(...)
>>>>>>> branch-a
```


