## 逆向解析：从零开始设计一个git

### 对git的头号疑问？

* **每次commit，Git储存的是全新的文件快照还是储存文件的变更部分？**





### git出现的原因？解决的问题？用来做什么？

* **linux**

版本管理 => 从上帝视角来设计一个系统

代码diff => 用优异的算法来生成
[Git 是怎样生成 diff 的：Myers 算法](https://cjting.me/2017/05/13/how-git-generate-diff/)



### git有哪些特点？

* **免费**：github
* **快**：git即便维护了超级大堆的文件，也非常快，随意版本切换，几乎感觉不到慢，凭什么
* **体积小**：维护了那么多版本，还感觉不到其空间的占用
* **多版本**：时间旅行式的版本管理
* **分布式**：多人协作，无网络也能提交代码



### 假如没有git，我们如何解决这些问题

* 初步设计git的接口




```typescript
// 文件：
interface file {
   content: string
   id: hash
   name: string
   auth: chmod
   parent: dir
   pathname: path
}

// 文件夹（文件树）：
interface dir {
    parent?: dir
    children:  <file | dir>[]
    id: hash
    name: string
    auth: chmod
}


// 提交 version
interface commit {
    author: string
    msg: string
    time: string
    id: hash
    version: dir
    parent: hash // commit
    children?: hash[] // commit[] // merge  // 后一个的追加不会影响前一个
}

// 仓库（版本链）：
type commit[]


```





* 版本管理 => 目标：不浪费空间，切版本不慢

  * 时间：不慢

  * 空间：
    一般情况下，版本之间，代码是渐变的，那么相同的部分该怎么利用起来以缩小空间占用呢

  * 最小单元：

    文件夹？文件？内容？提交（备注，内容）？版本？

    最小可复用单元：
    文件？内容？追踪文件？追踪内容？

    rename怎么办（只改了几个字符）? 权限更改了？

    内容更改了怎么办？

  * 提交

    改了提交信息怎么办？

    时间旅行 => 怎么实现时间旅行式的版本管理？指针？



```typescript
// 内容：
interface blob {
    id: hash
    content: string
}

// 内容树
interface tree {
    id: hash
    // parent?: dir
    children:  <blob | tree>[]
    auth: chmod
    name: string
}

// 提交 version
interface commit {
    id: hash
    version: tree
    parent: hash // commit
	  author: string
    msg: string
    time: string
    // children?: hash[] // commit[] // merge  // 后一个的追加不会影响前一个
}

// 单向链表
HEAD { id: hash }

{
  id: hash
  parent: hash
}

TAIL {
  id: hash
  parent: hash
}


```





#### git 三大区域

* **Working Directory （工作区）**

* **StagingArea(暂存区，索引区)** <blob | tree>[]

* **Repository（仓库区）** commit[]



#### git 三大类型

* **blob** 内容
* **tree** 内容树
* **commit** 提交



#### git 一个提交的流程

```shell
echo '111' > a.txt

git add a.txt // 将文件从工作区添加到暂存区

git ls-files // 列出当前索引
=>
100644 blob 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c	a.txt

git commit -m "init" // 将索引区的内容生成一个快照，以commit的形式存到仓库中
=>
[master (root-commit) 7a5d077] init
 1 files changed, 1 insertions(+)
 create mode 100644 a.txt

➜  gitLearn git:(master) ✗ git cat-file -t 7a5d077 # commit id
=>
commit

➜  gitLearn git:(master) ✗ git cat-file -p 7a5d077 # commit id
=>
100644 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c 0	a.txt

➜  gitLearn git:(master) ✗ git cat-file -t 58c9b # blob id
blob
➜  gitLearn git:(master) ✗ git cat-file -p 58c9b # blob id
111

```





```shell
➜  front-theory git:(bug/21244-msgfile) ✗ git log -1

commit c6dc74c0880891381e41540a2b898275eaf18eed (HEAD -> bug/21244-msgfile, origin/bug/21244-msgfile)
Author: zongze.li <zongze.li@q7link.com>
Date:   Thu Aug 27 12:47:42 2020 +0800

    <bug-21244>fix: 上传超过50M的文件并查看，跳转文件详情失败，直接退回到单据详情

➜  front-theory git:(bug/21244-msgfile) ✗ git cat-file -p c6dc74c0880891381e41540a2b898275eaf18eed

tree c3d3f232e10011aefa41ac850fc34f726ae0c798
parent 435ef1546afd65369368c618581dd643a7a3bd40
author zongze.li <zongze.li@q7link.com> 1598503662 +0800
committer zongze.li <zongze.li@q7link.com> 1598503662 +0800

<bug-21244>fix: 上传超过50M的文件并查看，跳转文件详情失败，直接退回到单据详情

➜  front-theory git:(bug/21244-msgfile) ✗ git cat-file -p c3d3f2

100644 blob 8bab3ff7089b109606ebf78ec7ad2e6c673e09c4	.dockerignore
100644 blob 530ca350dcbfb569b5abbf9a3721c1fcd34e2d61	.editorconfig
100644 blob 3045d637cd5acdecf03ab1acd4fbfa9f368d14dd	.gitignore
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	.npmrc
100644 blob a192ac33cf213e4122ba7ee6cf02403d96d8e326	.prettierrc
040000 tree 74843b1560caf70255e2875f2987812b075ffc87	.vscode
100644 blob 0c503dc4bc48a89af8915a0c56d2b43cd86bac39	CODEOWNERS
100644 blob 5ac0c30a870c3b2527ad55e47ec198fde593d454	Dockerfile
100644 blob 5d7bf828624550fc87b52a2eee49080cf9123f42	Dockerfile-nginx
100644 blob a000d1d6e4b5ec1a8eb3276e9d01c4500dc04c9a	README.md
100644 blob c9480f886c9989c0911e32f6c1f2710a3d13643e	TODO.md
040000 tree 4071a5b5819d1db48ded37913af46c5df35a1386	apps
040000 tree 06c9fd295add780dc3036ae6cd37e76c7f4d73c9	common
040000 tree 91e1b122aec9522dfe4b8ca72c9c88741579e0f6	config-sentry
040000 tree e06ce21bcb29c6674379683bfbc1e19d75c24834	config
100644 blob c85db377bbb0bce308730cb8e6d7f67d6fbe5e04	docker-compose-frontend.yml
100644 blob 67261bd3895200ee1cd277347ff6436756bdf93d	docker-compose.yml
100644 blob 534c1c425a0403245779681ff408bb2838b331b3	nginx.conf
100755 blob 40bba2686ca0ca7008d1bb2de9ce01ea2475c5d9	package-lock.json
100644 blob f08f94e4d0ab2511f7e61b71a9d53ece6acbb525	package.json
040000 tree 508c1dd7aae61ad716d64fc86733dddd20c569b9	packages
100644 blob 5c9b72b9f9156a5506a9eb4d5197dbc937800d16	rush.json
040000 tree 6da75061a216ceae957d9c34c607a307331429ff	scripts
100644 blob 2453a0f473ab882486ca888ebb9a7815bce22fe5	sh.exe.stackdump
100644 blob 6c70bcfe4d48d15f8a6531d6b491e65d641a377c	test.html
100644 blob 037cfac4db34127b6ecd9af0cb71cd7d320b53f0	tsconfig.json
```



```shell
➜  gitLearn git:(master) ✗ ls -R

a.txt c.txt g.txt src

./src:
b.txt  d.txt  level1

./src/level1:
level2

./src/level1/level2:
e.txt f.txt

➜  gitLearn git:(master) ✗ git ls-files

a.txt
c.txt
src/b.txt
src/d.txt
src/level1/level2/e.txt
src/level1/level2/f.txt

➜  gitLearn git:(master) ✗ git log -1

commit cebb2b58307845cd3486f0923ff42f66dbd0aa4a (HEAD -> master)
Author: zongze.li <zongze.li@q7link.com>
Date:   Thu Aug 27 15:08:50 2020 +0800

    tes src/level1/level2/f.txt

➜  gitLearn git:(master) ✗ git cat-file -p cebb2b # commit id

tree d9438663d50176bfb47f48af76a88a0d4f393471
parent d6b26264fa8bcd1e23c051e89c3e8afe7506d0e6
author zongze.li <zongze.li@q7link.com> 1598512130 +0800
committer zongze.li <zongze.li@q7link.com> 1598512130 +0800

tes src/level1/level2/f.txt

➜  gitLearn git:(master) ✗ git cat-file -p d94386 # tree id from commit

100644 blob 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c	a.txt
100644 blob 55bd0ac4c42e46cd751eb7405e12a35e61425550	c.txt
040000 tree 05efcc0e459116c913a0183b4615f5200bdee878	src

➜  gitLearn git:(master) ✗ git cat-file -p 05efcc # tree id from tree

100644 blob c200906efd24ec5e783bee7f23b5d7c941b0c12c	b.txt
100644 blob 1e6fd033863540bfb9eadf22019a6b4b3de7d07a	d.txt
040000 tree 08cdeda60cf55d54feb8c9743bfb9da2a1a90f5d	level1

➜  gitLearn git:(master) ✗ git cat-file -p 08cded # tree id from tree

040000 tree a49e1ff6ef362394a16200ae9df893e048500e30	level2

➜  gitLearn git:(master) ✗ git cat-file -p a49e1f # tree id from tree

100644 blob 3749383ded246790596d2f39db4dd8b71f525bbc	e.txt
100644 blob 7cc86ad136925c853a2ef916ae156927d1c06dfd	f.txt

```





~~？git commit 就是将暂存区打包成一个tree？~~

？git cherry-pick 怎么实现的





#### git 常用命令解读

* git add  => 将工作区的内容 添加到 暂存区 （在索引区生成索引）

* git reset => 将暂存区的文件 撤回到 工作区（在索引区删除对应索引）



* git commit => 将暂存区的文件提交到仓库（将索引区的内容生成一个快照）

* git reset --soft => 将仓库某个快照撤消掉，回到暂存区



* git reset --hard => 切换到某个快照 （并撤消当前所有改动）

* git checkout => 切换到某个快照 （将快照的文件取出来并和当前的改动进行merge）



#### git十大幻觉：

* 我一个人能解决merge时的冲突 * **10**


