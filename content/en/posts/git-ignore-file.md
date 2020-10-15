+++
title = "git忽略已经被提交的文件"
author = ["emacle"]
date = 2020-06-30T12:03:00+08:00
tags = ["git"]
draft = false
authorEmoji = "🎅"
image = "https://upload.wikimedia.org/wikipedia/commons/3/3f/Git_icon.svg"
pinned = false
+++

.gitignore 文件的用途，该文件只能作用于 Untracked Files，也就是那些从来没有被 Git 记录过的文件（自添加以后，从未 add 及 commit 过的文件）。

[How to ignore files already managed with Git locally](https://dev.to/nishina555/how-to-ignore-files-already-managed-with-git-locally-19oo)
方法一: git update-index

```sh
# 添加/确认/恢复
git update-index --skip-worktree path/to/file
git ls-files -v | grep ^S
git update-index --no-skip-worktree path/to/file
```

git ls-files　shows all files managed by git.
-v check the file being ignored.
--skip-worktree is displayed with S
--skip-worktree 不会提交改动的文件了，但是切换分支的话 git 还是要抱怨你的 worktree 不够干净。此时可以用 git stash 来存取解决这个问题

方法二: git update-index (`推荐，不用管是否切换分支`)

```sh
# 添加/确认/恢复
git update-index --assume-unchanged path/to/file
git ls-files -v | grep ^h  或 grep -E "^h"
git update-index --no-assume-unchanged path/to/file
```

Also, since it is an idea to ignore local changes, git reset - hard command will delete local changes.

```text
D:\Q\code\CTSC-Vue>git update-index --assume-unchanged vue.config.js
D:\Q\code\CTSC-Vue>git ls-files -v | grep.exe -E "^h "
h vue.config.js
```

<https://segmentfault.com/q/1010000000430426>
