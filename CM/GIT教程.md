# GIT教程



## 1. GIT序言

GIT是一种分布式的版本控制系统。一般将GIT结合代码审核服务器使用，这里结合```Gerrit```,  ```Git```， ```Jenkins```一起使用，```Git```用于管理代码，```Gerrit```用于代码审查，```Jenkins```用于编译。

使用GIT克隆，更新，提交代码的操作大致如下图绿色区域中的命令：

![1545796620546](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1545796620546.png)



上图中提交代码的流程大如下：

1. 从服务器克隆/更新项目到本地

2. 在工作目录中修改某些文件

3. 对修改后的文件进行快照，然后保存到暂存区域

4. 提交更新，将保存在暂存区域的文件快照永久转储到 Git 目录中

5. push提交到```Gerrit```缓存

6. Jenkins获取提交编译,审核者在Gerrit网页上审核代码，审核都通过提交到Git仓库中，审核失败开发者重新修改并重新提交

<font color="#FF4500">注意：</font>
如果是第一次使用该电脑克隆，修改提交该Gerrit服务器上的项目需要完成下面两个步骤：

1. 登录Gerrit，在设置中添加自己的邮箱（Settings->Contact Information->Register New Email）。

2. 在电脑上配置自己使用git提交的用户名，邮箱。

3. 在电脑上生成SSH公钥，秘钥，将公钥放到自己的Gerrit账户里面。



## 2. GIT安装

### 2.1 GIT在Windows中安装

在 Windows 上安装 Git 同样轻松，有个叫做 msysGit 的项目提供了安装包，可以到 GitHub 的页面上下载 exe 安装文件并运行：

```
https://git-scm.com/download/win
```

完成安装之后，就可以使用命令行的 `git` 工具（已经自带了 ssh 客户端）了，另外还有一个图形界面的 Git 项目管理工具。

### 2.2 GIT在Linux中安装

在 Fedora 上用 yum 安装：

```
$ yum install git-core
```

在 Ubuntu 这类 Debian 体系的系统上，可以用 apt-get 安装：

```
$ apt-get install git
```

### 2.3 GIT在Mac中安装

在 Mac 上安装 Git 有两种方式。

最容易的当属使用图形化的 Git 安装工具，界面如图 1-7，下载地址在：

```
http://sourceforge.net/projects/git-osx-installer/
```



![img](https://git-scm.com/figures/18333fig0107-tn.png)



另一种是通过 MacPorts (`http://www.macports.org`) 安装。如果已经装好了 MacPorts，用下面的命令安装 Git：

```
$ sudo port install git-core +svn +doc +bash_completion +gitweb
```

这种方式就不需要再自己安装依赖库了，Macports 会帮你搞定这些麻烦事。



## 2. GIT命令集

### 命令帮助文档

查看git命令帮助文档，下面的命令主要讲简单的用法，需要查看详细的使用说明就--help查看帮助文档

```
$ git --help
```
如果要查看某个子命令的帮助文档，git <subcommand> --help，for example.
```
$ git clone --help
```
此命令可以查看git clone 的帮助文档



### 公钥，私钥

Gerrit 服务器都使用 SSH 公钥进行认证，所以第一次在此电脑上从Gerrit(git服务器)克隆项目代码，需要生成公钥，私钥，并且拷贝到Gerrit上自己的账户中，使用下面命令生成公钥，私钥：
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/yangyanqiu/.ssh/id_rsa):
Created directory '/c/Users/yangyanqiu/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/yangyanqiu/.ssh/id_rsa.
Your public key has been saved in /c/Users/yangyanqiu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Pw6yPhmJZboj2d6X6xPEFB4zfpD+e3f8/+BCHD2I6r8 yangyanqiu@LAPTOP-GEU4A1P9
The key's randomart image is:
+---[RSA 2048]----+
|        *o       |
|       oo=       |
|       +o .. o   |
|      o +.. o o  |
|     = oSo . . . |
|    o o o.. o    |
|   o ..+.ooo  .. |
|  o +.oo=o..o...o|
|   o.o++o+Eo o..B|
+----[SHA256]-----+

```

Windows系统中默认会放在/c/Users/用户名/.ssh/中，Linux中默认放到/home/用户名/.ssh/中，里面的id_rsa.pub为公钥，id_rsa为私钥。

将新的公钥文件中的内容拷贝到Gerrit服务器中，登录进入Gerrit服务器，Settings->SSH Public Keys-> Add Key，拷贝公钥内容到这里然后保存，我们就可以从Gerrit服务器上克隆项目了:

![1545728260955](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1545728260955.png)

### 配置用户名邮箱地址

第一次在此电脑上使用git，并且需要提交东西，需要先配置好自己的用户名，邮箱地址：

```
$ git config --global user.name 'username'
$ git config --global user.email username@auto-link.com.cn
```

这里用户名一般使用自己的Gerrit账户，邮箱使用Gerrit账户中设置的邮箱。

### 克隆项目

可能已经注意到这里使用的是 `clone` 而不是 `checkout`。这是个非常重要的差别，Git 收取的是项目历史的所有数据（每一个文件的每一个版本），服务器上有的数据克隆之后本地也都有了。

```
$ git clone [url]
```
for example:
```
$ git clone ssh://username@192.168.3.7:29418/Test
```

这条命令在当前目录下创建一个名为`Test`的目录，里面包含项目中所有的文件，还有一个 `.git` 的目录，用于保存下载下来的所有版本记录。



### 查看提交历史

在克隆下来的项目目录中，使用下面命令可以查看日志：

```
$ git log
```



### 更新

更新服务上新的提交到本地可以使用git fetch或者git pull，区别是git pull = git fetch + git merge：

```
#从仓库下载所有的更新到本地
$ git fetch
```
或者

```
#从仓库下载所有的更新到本地，并且合并到当前分支
$ git pull
```



### 提交修改到暂存区

修改/新添加的文件必须先提交到暂存区（索引）中才能提交，下面命令为添加修改文件到索引中：

```
$ git add <fileName>
```

如果想直接一次添加所有修改的文件，可以使用git add -A，或者git add --all。



### 提交本地修改

```
$ git commit -m "Commit Message"
```

如果直接使用git commit，后面不接 -m, 这种方式会启动文本编辑器，然后直接在文本编辑器中输入本次提交的说明，输入完保存退出就好。使用 `git config --global core.editor` 命令设定你喜欢的编辑软件。



### 查看文件状态

使用git status查看工作区文件的状态：

```git
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   b.cpp

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   readme

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        a.cpp
```

文件有三种状态：

已修改（modified）：已修改表示修改了某个文件，但还没有提交保存。

已暂存（staged）：已暂存表示把已修改的文件放在下次提交时要保存的清单中。

已提交（committed）：已提交表示该文件已经被安全地保存在本地数据库中了。

上图中未跟踪（Untracked），未缓存（not staged）的文件都属于已修改；将被提交（Changes to be committed）的文件属于已暂存的文件。git add命令会暂存已修改的文件，git commit会提交所有暂存了的文件。



### 比较两次修改的差异

使用git diff比较工作目录，暂存区，版本库中文件/提交之间的差异：


```
#不加参数即比较工作目录，暂存区之间的差异
$ git diff

#比较工作区与最新的版本库
$ git diff HEAD [<path>...]

#比较两次提交之间的差异
$ git diff [<commit-id>] [<commit-id>]

#比较暂存区与最新的本地版本库
$ git diff --cached [<path>...]
...
```
最常用的是比较工作目录与版本库之前的差异



### 推送本地提交到远程仓库

```
#git push <远程主机名> <本地分支名>:<远程分支名>
$ git push origin HEAD:refs/for/<branchName>
```

origin：表示远程主机，可以使用git remote查看，默认远程主机都是origin
HEAD：指向当前分支
远程分支：使用```Gerrit```做代码审查，开发者仅有权限提交到```Gerrit```缓存，即分支为refs/for/<branchName>



### 创建、删除标签

标签（Tag）其实就是指向某个commit的指针。发布一个版本时，通常先在版本库中打一个标签（tag），方便随时可以回到这个版本。Git 使用的标签有两种类型：轻量级的（lightweight）和含附注的（annotated）。

1. 查看现有标签

```
$ git tag
```

1. 创建标签

```
#创建轻量级的标签v0.8指向commitID
$ git tag <tagName> <commitID>
#创建含附注的标签：标签名为v0.9
$ git tag -a <tagName> -m 'Commit Message' <commitID>
#for example: 
$ git tag -a v0.9 -m 'version v0.9' cae81b142a8cd28b06bd851c0b7f2cb5c1399be8
```

1. 推送标签到Gerrit服务器

```
$ git push origin <tagName>:refs/tags/<tagName>
```

1. 删除标签

```
#删除本地标签
$ git tag -d <tagName>
#删除服务器标签
$ git push origin :refs/tags/<tagName>
```

轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。

含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用GNU Privacy Guard (GPG) 来签署或验证。建议使用含附注型的标签，以便保留相关信息。



### 创建本地分支

git branch用于创建，删除，列出分支， git checkout也可以用于创建分支：

```
#创建分支
$ git branch <branchName>
#创建分支，并且切换到刚创建的分支
$ git checkout -b <branchName>

#列出本地所有分支
$ git branch
#删除分支
$ git branch -d <branchName>
```



### 切换分支

使用git checkout切换分支

```
$ git checkout <branchName>
```



### 合并分支

合并分支可以使用git merge, git rebase。git merge会生成一个新的提交，并且将之前的提交按照提交时间排序；git rebase不会生成新的提交，只是保持当前分支提交不变，然后把要合并的分支的提交放到当前分支最新提交的后面。

```
#将远程分支merge到当前分支
$ git merge b
#rebase同样可以将b分支merge到当前分支
$ git rebase b

```
下图为当前提交状态，当前分支未dev，提交的C3，C4，C5，C6的提交时间分别为9:00AM， 16:00PM, 9:30AM, 10:00AM：

![1545722404932](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1545722404932.png)



1. Merge: 使用git pull将把"origin"分支上的修改拉下来并且和你的修改合并，合并后的dev分支排序如下图：

![1545722614343](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1545722614343.png)

如果merge遇到冲突，解决冲突文件后重新提交冲突文件：

```
#解决冲突文件后
$ git add 冲突文件
$ git rebase --continue
```



2. Rebase: 使用git fetch先把远程分支新的提交C3，C4获取到本地，然后git rebase origin将远程分支合并到dev分支，合并如下图：

![1545722797151](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1545722797151.png)

说明：

git rebase合并分支时，dev分支里的提交(C4，C5)会被取消掉，并且把它们临时保存为补丁(patch)(这些补丁放到".git/rebase"目录中)，然后把"dev"分支更新为最新的"origin"分支，最后把保存的这些补丁应用到"dev"分支上。

如果rebase遇到冲突，解决冲突后使用下面方式继续rebase：

```
#解决冲突文件后
$ git add 冲突文件
$ git rebase --continue
```



### 回退版本

把当前的HEAD重置到以前的某个状态，错误commit之后，想恢复到某个版本库的代码：

```
#回退到commitID那个提交,清空暂存区,工作目录的内容，工作目录的文件将被恢复的版本替换
$ git reset --hard commitID
#回退到commitID那个提交，将暂存区的内容和已提交的内容全部恢复到未暂存的状态,不影响原来工作目录的文件(未提交的也不受影响) 
$ git reset --mixed commitID
#回退到commitID那个提交,不清空暂存区，将已提交的内容恢复到暂存区,不影响原来工作目录的文件(未提交的也不受影响) 
$ git reset --soft commitID
```



## 3. 实例

从Gerrit上克隆Test项目，添加一个文件，并且提交到Gerrit服务器master分支，步骤如下：

```
#step1. 生成公钥密码，输入下面命令一直回车，如果提示已经存在就不用重新生成
$ ssh-keygen
#step2. 登录Gerrit, 将id_rsa.pub文件中的内容拷贝到SSH Public Keys中
$ 手动将id_rsa.pub文件中的内容拷贝到SSH Public Keys
#step3. 配置本地提交提交账户，用户名
$ git config --global user.name yangyanqiu
$ git config --global user.email yangyanqiu@auto-link.com.cn
######上面几个步骤一般只需要做一次##########################

#使用的yangyanqiu的账号克隆
$ git clone ssh://yangyanqiu@192.168.3.7:29418/Test
#创建temp.txt文件
$ touch temp.txt
#将新创建的temp.txt添加到缓存中
$ git add temp.txt
#提交已经缓存了的内容
$ git commit -m "chore:add temp file named temp.txt"
#将本地的提交推送到Gerrit服务器中，第一次提交可能会出现下面错误
$ git push origin HEAD:refs/for/master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 279 bytes | 279.00 KiB/s, done.
Total 2 (delta 0), reused 0 (delta 0)
remote: Processing changes: refs: 1, done
remote: ERROR: [a089096] missing Change-Id in commit message footer
remote:
remote: Hint: To automatically insert Change-Id, install the hook:
remote:   gitdir=$(git rev-parse --git-dir); scp -p -P 29418 yangyanqiu@192.168.3.7:hooks/commit-msg ${gitdir}/hooks/
remote: And then amend the commit:
remote:   git commit --amend
remote:
To ssh://192.168.3.7:29418/Test
 ! [remote rejected] HEAD -> refs/for/master ([a089096] missing Change-Id in commit message footer)
error: failed to push some refs to 'ssh://yangyanqiu@192.168.3.7:29418/Test'

```

克隆下来的项目，第一次推送本地提交到Gerrit会报上面的错，错误信息为“ERROR: missing Change-Id in commit message footer”，提交信息页脚缺少了Change-Id，造成错误的原因是缺少了服务器上某些钩子(hooks)。

解决方法：需要从服务器上把钩子获取到本地，重新提交，根据错误信息里面的提示执行下面命令即可：

```
#下面命令里面的yangyanqiu为自己的账户
$ gitdir=$(git rev-parse --git-dir); scp -p -P 29418 yangyanqiu@192.168.3.7:hooks/commit-msg ${gitdir}/hooks/
#为之前的提交生成Change-Id
$ git commit --amend --no-edit
#重新提交
$ git push origin HEAD:refs/for/master
```