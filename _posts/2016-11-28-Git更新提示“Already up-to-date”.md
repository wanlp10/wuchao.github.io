---
layout: post
title: Git 更新提示 “Already up-to-date”
category : 学习问题记录
tagline: "Supporting tagline"
tags : [Git]
---
{% include JB/setup %}
# Git 更新提示 “Already up-to-date”
---

在使用 git 的过程中由于误操作，导致从本地 master 分支 merge 代码到当前分支失败，虽然当前分支和 master 分支代码不同步，但是仍然提示 `Already up-to-date`。

参考 StackOverflow 上面的解答：
> [https://stackoverflow.com/questions/634546/git-merge-reports-already-up-to-date-though-there-is-a-difference](https://stackoverflow.com/questions/634546/git-merge-reports-already-up-to-date-though-there-is-a-difference)
  
大体意思是说当前分支已经同本地 master 分支代码合并过了，并且本地 master 分支代码已经是最新的了。  

这时可以使用 `git reset --hard` 命令将本地防止代码回滚到以前某一个正确的版本，再配合 `git push --force` 命令将本地分支的修改同步到服务器上，不然服务器上的分支又会和回滚后的分支冲突：  

```
git reset --hard <version>
git push origin <branch> --force
```

`<version>` ：表示将要回滚到的当前分支的历史版本号。  
`<branch>` ：表示当前的本地分支名称。