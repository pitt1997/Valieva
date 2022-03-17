# git简介

git是目前流行的分布式版本管理系统。它拥有两套版本库，本地库和远程库，在不进行合并和删除之类的操作时这两套版本库互不影响。也因此其近乎所有的操作都是本地执行，所以在断网的情况下任然可以提交代码，切换分支。

<img src="images\image-20220317221346398.png" alt="image-20220317221346398" style="zoom:80%;" />



# git优势

集中式版本控制（SVN）只有中心服务器拥有一份代码，而分布式版本控制每个人的电脑上就有一份完整的代码。

集中式版本控制有安全性问题，当中心服务器挂了所有人都没办法工作了。

集中式版本控制需要连网才能工作，如果网速过慢，那么提交一个文件会慢的无法让人忍受。而分布式版本控制不需要连网就能工作。

分布式版本控制新建分支、合并分支操作速度非常快，而集中式版本控制新建一个分支相当于复制一份完整代码。



# git工作流

新建一个仓库之后，当前目录就成为了工作区，工作区下有一个隐藏目录 .git，它属于 Git 的版本库。

Git 的版本库有一个称为 Stage 的暂存区以及最后的 History 版本库，History 存储所有分支信息，使用一个 HEAD 指针指向当前分支。

- git add files 把文件的修改添加到暂存区
- git commit 把暂存区的修改提交到当前分支，提交之后暂存区就被清空了
- git reset -- files 使用当前分支上的修改覆盖暂存区，用来撤销最后一次 git add files
- git checkout -- files 使用暂存区的修改覆盖工作目录，用来撤销本地修改



<img src="images\image-20220317221741464.png" alt="image-20220317221741464" style="zoom:80%;" />

可以跳过暂存区域直接从分支中取出修改，或者直接提交修改到分支中。

- git commit -a 直接把所有文件的修改添加到暂存区然后执行提交
- git checkout HEAD -- files 取出最后一次修改，可以用来进行回滚操作

<img src="images\image-20220317221834915.png" alt="image-20220317221834915" style="zoom:80%;" />



# Git 命令一览

<img src="images\image-20220317222130303.png" alt="image-20220317222130303" style="zoom: 80%;" />

# 参考

https://github.com/CyC2018/CS-Notes/blob/master/notes/Git.md

http://marklodato.github.io/visual-git-guide/index-zh-cn.html

