## git 常用操作集锦
————————新手必读

【此链接必读】请下面这个链接的内容非常重要，Git 的思想和基本工作原理：
https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-Git-%E5%9F%BA%E7%A1%80

若你理解了 Git 的思想和基本工作原理，用起来就会知其所以然，游刃有余。
这个链接解释了git 版本控制的基本工作原理，
基础原理理解了，git 那些命令信手拈来哈

————————看下面这些常用git命令之前，先看上面的链接哈（老司机请忽略）

```shell
git status 查看当前工作区的状态，哪些文件修改了，add了，删除了，新增了

git add / git reset

git log --author --grep --no-merges

git diff 查看代码的修改情况

git commit -m 提交

git pull （等效于git fetch(拉取) + git merge(合并)）/ git rebase

git push 推送

git pull origin branchName 拉取远程分支

git cherry-pick 38361a68  将已经提交的commit中的修改, 应用到其它分支上。（同时维护几个分支的神器啊！！）

git revert -m 1 08932325f580553a581b1590f07ca2ff7dcb54cb  回退 merge之类的

git revert tag
git reset --hard tagname

git merge --abort 一般用于退出merge，(merge不知归路时，想放弃时用)
```



#### git add (添加到缓存区)

```shell
git add xx命令可以将xx文件添加到暂存区

git add -A .如果有很多改动可以通过 git add -A .来一次添加所有改变的文件。注意 -A 选项后面还有一个句点。
git add -A 表示添加所有内容，
git add . 表示添加新文件和编辑过的文件不包括删除的文件;
git add -u 表示添加编辑或者删除的文件，不包括新添加的文件。
git add path  将某个路径下的所有修改的文件提交
```



#### git reset

重置：将提交到缓存区的修改撤回到编辑工作区，回退

重置的命令与add命令是配套的，操作可逆性

```shell
git reset . / git add .
git rest *
git reset fileName
git reset path  将某个路径下的所有提交的文件重置 支持路径匹配和匹配符
```



回退：

```shell
git reset --hard commitId(08932325f580553a581b1590f07ca2ff7dcb54cb 或者前6位数字也行)
搭配 git log 穿越回溯分分钟

git reset --soft HEAD^：将最近一次提交节点的提交记录回退到暂存区
git reset --mixed HEAD^：将最近一次提交节点的提交记录回退到工作区
git reset --hard HEAD^：将最近一次提交节点的提交记录全部清除
```



#### git diff

```shell
git diff 查看未add的修改
git diff --cached 查看已add的修改
git diff fileName
git diff path
Git diff branch1 branch2 --stat   //显示出所有有差异的文件列表
Git diff branch1 branch2 文件名(带路径)   //显示指定文件的详细差异
Git diff branch1 branch2                   //显示出所有有差异的文件的详细差异

这个是 diff 两个 commit 之间的内容, 是 commit 的 ID 号，可以通过 git log 查看。获取到两个 commit 的 ID 号后，就可以查看两个 commit 的修改内容了。转载请注明来自：http://www.binkery.com/
git diff commit-id-1 commit-id-2
git diff ./ ":(exclude)*.json"
```



#### git stash

```shell
git stash  对当前的暂存区和工作区状态进行保存。
git stash list  列出所有保存的进度列表。
git stash pop [--index] [<stash>] 恢复工作进度
git stash apply
git stash drop
一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；
另一种方式是用git stash pop，恢复的同时把stash内容也删了：

git stash pop --index stash@{0}
```



#### git checkout

```shell
git checkout . 撤消所有当前环境的修改，不可恢复
git checkout fileName 撤消该文件的修改
git checkout branchName 切换分支
git checkout - 切换到上一个分支
git checkout -b branchName 新建分支
```



#### git branch

```shell
git branch 查看本地所有分支
git branch -a 查看本地和远程所有分支
git branch -D branchName 删除本地分支
```



#### git config

用alias简化命令
git config --global alias.co checkout
git config --global alias.br branch

```shell
1. 配置使用git仓库的人员姓名
2. git config --global user.name "Your Name Comes Here"
3.
4. 配置使用git仓库的人员email
5. git config --global user.email you@yourdomain.example.com
6.
7. 配置到缓存 默认15分钟
8. git config --global credential.helper cache
9.
10. 修改缓存时间
11. git config --global credential.helper 'cache --timeout=3600'
12.
13. git config --global color.ui true
14. git config --global alias.co checkout
15. git config --global alias.cm commit -m # git cm ""
16. git config --global alias.st status # git st
17. git config --global alias.br branch # git br
18. git config --global core.editor "mate -w"    # 设置Editor使用textmate
19. git config -1 #列举所有配置
20.
21. 用户的git配置文件~/.gitconfig
```



忽略大小写
git config --global core.ignorecase false



### git 奇淫技巧

 git for-each-ref --format='%(committerdate) %09 %(authorname) %09 %(refname)' | sort -k5n -k2M -k3n -k4n | grep <branch>

cat .git/refs/heads/<branch>



####  keep git log

* How to keep git log and less output on the screen

git config --global --replace-all core.pager "less -iXFR"



#### 修改commit信息

另外很重要的一点是：  当我们执行commit命令之后发现有文件未添加或是commit message书写有错误，如果还未push到远端，那么可以使用下面方法进行处理。如果已经push到远端，那么其就无法被修改。

```shell
# 添加未被添加到暂存区的文件
# 然后使用--amend
git add [filename]
git commit --amend

# 会自动加上已经修改但未被添加到暂存区的文件
git commit --amend -a
```

￼￼￼￼￼
执行后可看到类似下图的结果，可直接修改信息并回车保存

￼
也可使用reset恢复版本，然后重新提交。



#### 批量处理冲突
git checkout --ours 和 git checkout --theirs



#### HEAD

HEAD~
HEAD^
换成~：git reset --hard HEAD~ 或者 git reset --hard HEAD~1
~ 后面的数字表示回退几次提交，默认是一次

#### 显示本地操作记录

* git reflog

当commit后，未push又reset —hard 回去了，找不到当前的commit id时，用git reflog看操作记录就能找到了



#### 删除远端分支

* git push origin :[branch] 删除远端分支

* git remote prune origin 删除远端已经删除的分支



#### git rebase

重写历史: 修改多个提交说明

```shell
$ git rebase -i HEAD~3
再次提醒这是一个衍合命令——HEAD~3..HEAD范围内的每一次提交都会被重写，无论你是否修改说明。不要涵盖你已经推送到中心服务器的提交——这么做会使其他开发者产生混乱，因为你提供了同样变更的不同版本。
运行这个命令会为你的文本编辑器提供一个提交列表，看起来像下面这样
pick f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file

Rebase 710f0f8..a5f4a0d onto 710f0f8

Commands:
 p, pick = use commit
 e, edit = use commit, but stop for amending
 s, squash = use commit, but meld into previous commit
 If you remove a line here THAT COMMIT WILL BE LOST.
 However, if you remove everything, the rebase will be aborted.

很重要的一点是你得注意这些提交的顺序与你通常通过log命令看到的是相反的。如果你运行log，你会看到下面这样的结果：
git log --pretty=format:"%h %s" HEAD~3..HEAD
a5f4a0d added cat-file
310154e updated README formatting and added blame
f7f3f6d changed my name a bit
请注意这里的倒序。交互式的rebase给了你一个即将运行的脚本。它会从你在命令行上指明的提交开始(HEAD~3)然后自上至下重播每次提交里引入的变更。它将最早的列在顶上而不是最近的，因为这是第一个需要重播的。
你需要修改这个脚本来让它停留在你想修改的变更上。要做到这一点，你只要将你想修改的每一次提交前面的pick改为edit。例如，只想修改第三次提交说明的话，你就像下面这样修改文件：
edit f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
当你保存并退出编辑器，Git会倒回至列表中的最后一次提交，然后把你送到命令行中，同时显示以下信息：
$ git rebase -i HEAD~3
Stopped at 7482e0d... updated the gemspec to hopefully work better
You can amend the commit now, with

       git commit --amend

Once you’re satisfied with your changes, run

       git rebase --continue

这些指示很明确地告诉了你该干什么。输入
 git commit --amend
修改提交说明，退出编辑器。然后，运行
 git rebase --continue

导致无法正常切换：error: The following untracked working tree files would be overwritten by checkout
通过错误提示可知，是由于一些untracked working tree files引起的问题。所以只要解决了这些untracked的文件就能解决这个问题。
```



#### git clean 删除多余文件

    git clean 参数

    -n 显示将要删除的文件和目录；

    -x -----删除忽略文件已经对git来说不识别的文件

    -d -----删除未被添加到git的路径中的文件

    -f -----强制运行

git clean -d -f
git reset --hard

应该一般建议使用 git clean -d -f

使用git clean -d -fx会把一些程序必要的文件删除，会把git本来ignore的文件删除。



#### git remote

git remote -v
git remote set-url origin [url]
git remote prune origin 删除远端已经删除的分支


