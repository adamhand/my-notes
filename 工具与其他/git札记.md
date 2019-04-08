# git札记

---
## 1. 安装git
略
安装完之后需要设置一步，因为Git是分布式版本控制系统，每个机器都必须有自己的地址和名字。
```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
其中`git config`命令的'--global'参数表示这台机器上所有的git仓库都会使用这个配置，但是也可以对某个仓库指定不同的用户名和Email地址。
## 2. 创建版本库
版本库又名仓库，英文名repository，可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。
创建版本库需要以下几个步骤：
> 1.新建目录
> 2.使用`git init`命令把这个目录变成Git可以管理的仓库。
```
$ mkdir learngit
$ git init
Initialized empty Git repository in /***/.git/
```
把文件放到Git仓库只需要两步
第一步，用命令`git add`告诉Git，把文件添加到仓库。
第二步，用命令`git commit`告诉Git，把文件提交到仓库。
```
$ git add readme.txt
$ git commit -m "wrote a readme file"
[master (root-commit) eaadf4e] wrote a readme file
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
```
`git commit`命令，`-m`后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样就能从历史记录里方便地找到改动记录。
`commit`可以一次提交很多文件，所以可以多次`add`不同的文件，比如
```
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."
```
## 3. 时光穿梭机
### 3.1 版本回退
在Git中，可以用`git log`命令查看历史记录。`git log`命令显示从最近到最远的提交日志。
```
$ git log
commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:06:15 2018 +0800

    append GPL

commit e475afc93c209a690c39c13a46716e8fa000c366
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:03:36 2018 +0800

    add distributed

commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 20:59:18 2018 +0800

    wrote a readme file
```
如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数：
```
$ git log --pretty=oneline
1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
e475afc93c209a690c39c13a46716e8fa000c366 add distributed
eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file
```
版本回退可以使用`reset`命令。下面的命令可以回到上一个版本。
```
$ git reset --hard HEAD^
HEAD is now at e475afc add distributed
```
下面的命令可以回退或前进到指定版本。
```
$ git reset --hard 1094a
HEAD is now at 83b0afe append GPL
```
![](https://raw.githubusercontent.com/adamhand/learngit/master/git%20head.jpg)
Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把`HEAD`从指向`append GPL`(上一个提交的版本)。
可以使用`git reflog`命令查看之前的每一次命令，可以用来查询commit id，方便回退查询。
```
$ git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file
```
### 3.2 工作区和暂存区
Git和其他版本控制系统如SVN的一个不同之处就是有暂存区的概念。
<br/>
**工作区（Working Directory）**
就是在电脑里能看到的目录，比如learngit文件夹就是一个工作区：
![](https://raw.githubusercontent.com/adamhand/learngit/master/learngit.png)
**版本库（Repository）**
工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。
Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。
![](https://raw.githubusercontent.com/adamhand/learngit/master/repository.jpg)
我们把文件往Git版本库里添加的时候，是分两步执行的：
第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；
第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。
因为我们创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit` 就是往`master`分支上提交更改。
可以用`git status`命令查看一下暂存区的状态：
```
$ git status
```
### 3.3 撤销修改
命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：
一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是**让这个文件回到最近一次`git commit`或`git add`时的状态**。
所以，存在以下几种情况（*工作区、暂存区、版本库、远程仓库*）：
>场景1：当你改乱了工作区某个文件的内容，还没使用`git add`添加到暂存区时， 想直接丢弃工作区的修改时，用命令git checkout -- file。(注意`--`和前面和后面都有空格)
场景2：当你不但改乱了工作区某个文件的内容，还使用`git add`命令添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。
场景3：已经使用`git commit`命令提交了不合适的修改到版本库时，想要撤销本次提交，参考*版本回退*一节，不过前提是没有推送到远程库。
### 3.4 删除文件
当手动删除或者使用`rm test.txt`命令删除`test.txt`文件之后，有两种选择：
> 1. 确实想删除改文件，则可以使用如下命令在版本库中删除:
```
$ git rm test.txt
rm 'test.txt'

$ git commit -m "remove test.txt"
[master d46f35e] remove test.txt
 1 file changed, 1 deletion(-)
 delete mode 100644 test.txt
```
> 2. 删错了，想恢复文件，可以使用`git checkout`命令（`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。）：
```
$ git checkout -- test.txt
```
## 4. 远程仓库
### 4.1 添加远程仓库
详见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013752340242354807e192f02a44359908df8a5643103a000)
在连接github远程仓库的时候可能会遇到ssh key的问题，可以参考下面的博客配置ssh key：[配置ssh key](https://blog.csdn.net/lqlqlq007/article/details/78983879)
添加远程仓库时比较常用的 两个命令是：
```
$ git remote add origin git@github.com:xxx/xxx.git
```
添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
```
$ git push -u origin master
```
这个命令就可以把本地库的所有内容推送到远程库上，实际上是把当前分支`master`推送到远程。由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令，不使用`-u`参数。
需要注意的是，`git push`命令是将*本地仓库*中的内容推送到*远程仓库*，所以必须是`commit`之后的内容才能推送上去。
### 4.2 从远程仓库克隆
主要命令：
```
$ git clone git@github.com:xxx/xxx.git <本地目录名>
```
如果不指定目录的话，windows系统就会默认下载到`/c/Users/Administrator/.ssh/`;linux系统就会默认下载到`/home/`。
详见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375233990231ac8cf32ef1b24887a5209f83e01cb94b000)

## 5. 分支管理
### 5.1 创建与合并分支
具体参见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000)
常用命令：
> 查看分支：`git branch`
创建分支：`git branch <name>`
切换分支：`git chechout <name>`
创建+切换分支：`git checkout -b <name>`
合并某分支到当前分支：`git merge <name>`
删除分支：`git branch -d <name>`

示例：
```
$ git checkout -b dev          //创建分支dev并进行切换，该命令可由下列两条命令代替。
$ git branch dev               //创建dev分支
$ git checkout dev             //切换到dev分支

$ git branch                   //查看当前分支
* dev
  master

$ git add readme.txt           //在dev分支中提交
$ git commit -m "branch test"
[dev b17d20e] branch test
 1 file changed, 1 insertion(+)
 
 $ git checkout master          //切换到master分支
 
 $ git merge dev                //把dev分支的工作成果合并到master分支上
Updating d46f35e..b17d20e
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
 
 $ git branch -d dev            //删除dev分支
Deleted branch dev (was b17d20e).
```
### 5.2 解决冲突
当在两个分支上修改同一个文件并提交时可能会产生冲突，需要手动修改文件解决冲突。具体参见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840202368c74be33fbd884e71b570f2cc3c0d1dcf000)
使用`git log --graph --pretty=oneline --abbrev-commit`命令可以查看分支合并情况
```
$ git log --graph --pretty=oneline --abbrev-commit
*   cf810e4 (HEAD -> master) conflict fixed
|\  
| * 14096d0 (feature1) AND simple
* | 5dc6824 & simple
|/  
* b17d20e branch test
* d46f35e (origin/master) remove test.txt
* b84166e add test.txt
* 519219b git tracks changes
* e43a48b understand how stage works
* 1094adb append GPL
* e475afc add distributed
* eaadf4e wrote a readme file
```

### 5.3 分支管理策略
具体参见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013758410364457b9e3d821f4244beb0fd69c61a185ae0000)
合并分支时，加上`--no-ff`参数就强制禁用`Fast forward`模式，Git就会在`merge`时生成一个新的`commit`，就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。
示例：
```
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```

### 5.4 bug分支
具体参见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001376026233004c47f22a16d1f4fa289ce45f14bbc8f11000)
修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；
当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场。
示例：需要将`readme.txt`文件中的`git is free software`改成`git is a free software`
> 首先，把当前工作现场“储藏”起来
```
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```
现在，用·git status·查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。
> 确定要在哪个分支上修复bug，假定需要在`master`分支上修复，就从`master`创建临时分支
```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```
> 修复bug后提交
```
$ git add readme.txt 
$ git commit -m "fix bug 101"
[issue-101 4c805e2] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```
> 修复完成后，切换到`master`分支，并完成合并，最后删除`issue-101`分支

```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
 readme.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

$ git branch -d issue-101
```
> 切换到dev工作区继续工作，并使用`git stash list`命令查看工作现场
```
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```
> 恢复现场，有两个办法：一是用`git stash apply`恢复，但是恢复后，`stash`内容并不删除，你需要用`git stash drop`来删除；另一种方式是用`git stash pop`，恢复的同时把`stash`内容也删了
```
$ git stash apply
$ git stash drop

$ git stash pop
```
可以多次`stash`，恢复的时候，先用`git stash list`查看，然后恢复指定的`stash`，用命令
```
$ git stash apply stash@{0}
```
### 5.5 feature分支
具体参见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001376026233004c47f22a16d1f4fa289ce45f14bbc8f11000)
开发一个新功能，最好新建一个feature分支；
如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。

### 5.6 多人协作
具体参见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013760174128707b935b0be6fc4fc6ace66c4f15618f8d000)
> *多人协作的工作模式通常是这样：*
1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！
如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。
这就是多人协作的工作模式，一旦熟悉了，就非常简单。

> *小结*
查看远程库信息，使用`git remote -v`；
本地新建的分支如果不推送到远程，对其他人就是不可见的；
从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用git pull抓取远程的新提交；
在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

## 6.其他
详见[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)