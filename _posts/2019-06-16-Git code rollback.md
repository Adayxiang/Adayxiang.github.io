---
layout:     post                       # 使用的布局（不需要改）
title:      Git code rollback               # 标题 
subtitle:   correct posture of the rollback code #副标题
date:       2019-06-16                 # 时间
author:     adayxiang                         # 作者
header-img: img/post-bg-debug.png     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
    - Mac
    - Terminal
    - Git
---

> Personal documents that are not suitable for reading.

The difference between # **git revert** and **git reset**
 Look at the picture first:
 
![](https://ww3.sinaimg.cn/large/006tNbRwgy1fcr9tu6vdjj30t30ez0y8.jpg)

**sourceTree** **revert** translates to **`submit rollback`**, which ignores the version you specify and submits a new version. The version you specified has been removed in the new version.

**reset** resets the content to the specified version for **reset to this commit**. The `git reset` command is followed by two arguments: `–-hard` and `–-soft`. This command is `-–soft` by default.

When the above command is executed, all commit changes after the commit number (time as a reference point) will be returned to the git buffer. These changes can be seen in the buffer using the `git status` command. If you add the `-–hard` parameter, the changes will not be stored in the buffer, and git will discard this part directly. You can force the contents of the partition to be pushed to the remote server using `git push origin HEAD --force`.


#### Code fallback

The default parameter `-soft`, all commit changes will be returned to the git buffer
Parameter `--hard`, all commit changes are discarded directly

$ git reset --hard HEAD^ Fall back to the previous version
$ git reset --hard commit_id Back to / Go to Specify commit_id
Push to remote

$ git push origin HEAD --force


#### Can regret the medicine -> version shuttle

When you roll back, regret it again, what if you want to revert to the new version?

Print your record of each operation with `git reflog`

$ git reflog

Output:
C7edbfe HEAD@{0}: reset: moving to c7edbfefab1bdbef6cb60d2a7bb97aa80f022687
470e9c2 HEAD@{1}: reset: moving to 470e9c2
B45959e HEAD@{2}: revert: Revert "add img"
470e9c2 HEAD@{3}: reset: moving to 470e9c2
2c26183 HEAD@{4}: reset: moving to 2c26183
0f67bb7 HEAD@{5}: revert: Revert "add img"

Find the id of your operation such as `b45959e`, you can fall back to this version.

$ git reset --hard b45959e
