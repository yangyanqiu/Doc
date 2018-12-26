# SVN安装与使用

## 1.SVN简介

SVN是Subversion的简称，是一个开放源代码的版本控制系统。使用SVN管理文档的优势：

1. 防止文档丢失。
2. 查看提交日志可以看到别人做了什么修改。
3. 权限控制



## 2. SVN安装

### 2.1 SVN在Windows上安装

下载TortoiseSVN，下载地址：https://www.visualsvn.com/visualsvn/download/tortoisesvn/

双击文件安装。

在电脑上随便找个空白的地方单击右键出现下图则说明安装成功：

### 2.2 SVN在Ubuntu上安装

在Ubuntu上直接使用下面命令安装：

$ sudo apt-get install subversion

## 3. SVN的使用

### 3.1 在windows上使用

1. 检出项目

   在空白处右键->SVN Checkout。

   填写仓库URL，要检出到本地的路径，检出深度：

![检出对话框](http://svndoc.iusesvn.com/tsvn/1.5/images/Checkout.png)

可以选择检出的深度，默认都选择Fully recursive:

Fully recursive

检出完整的目录，包含所有的文件或子目录。

Immediate children,including folders

检出目录，包含其中的文件或子目录，但是不递归展开子目录。

Only File children

检出指定目录，包含所有文件，但是不检出任何子目录。

Only This Item

只检出目录。不包含其中的文件或子目录。

2. 更新工作副本

进入到项目，在空白处右键->SVN Update之后就会开始从服务器上更新最新的修改到本地。

![用户可以并行的工作，不必等待别人，当工作在同一个文件上时，也很少会有重叠发生，冲突并不频繁，处理冲突的时间远比等待解锁花费的时间少。已经完成更新的进度对话框](http://svndoc.iusesvn.com/tsvn/1.5/images/UpdateFinished.png)

进度对话框使用颜色代码来高亮不同的更新行为

- 紫色

  新项已经增加到你的工作副本中。

- 深红

  你的工作副本中删除了多余项，或是你的工作副本中丢失的项被替换。

- 绿色

  版本库中的修改与你的本地修改成功合并。

- 亮红

  来自版本库的修改在与本地修改合并时出现了冲突，需要你解决。

- 黑色

  你WC中的没有改动的项被来自版本库中新版本所更新。


3. 查看日志

TortoiseSVN->Show log可以查看提交的日志

4. 提交修改

将你对工作副本的修改发送给版本库，称为*提交*修改。但在你提交之前要确保你的工作副本是最新的，你可以直接使用TortoiseSVN → Update，然后再提交修改TortoiseSVN → Commit：

![提交对话框](http://svndoc.iusesvn.com/tsvn/1.5/images/Commit.png)

Message（提交说明）：一定要填写，这个会出现在日志中，说明一定要写清楚，方便查看日志知道此次提交修改了什么。

提交对话框将显示每个被改动过的文件，包括新增的、删除的和未受控的文件。如果你不想改动被提交，只要将该文件的复选框的勾去掉就可以了。如果你要加入未受控的文件，只要勾选该文件把它加入提交列表就可以了。



5. 撤销更改

如果你想要撤消一个文件自上次更新后的所有的变更，你需要选择该文件, 右击弹出快捷菜单，然后选择TortoiseSVN → Revert命令，将会弹出一个显示这个你已经变更并能恢复的文件，选择那些你想要恢复的然后按OK. 

![恢复对话框](http://svndoc.iusesvn.com/tsvn/1.5/images/Revert.png)



6. 查看文件修改

选中文件，TortoiseSVN->diff可以查看本地修改内容。

![1544442013627](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1544442013627.png)

左边是版本18文件的内容

右边是修改后的内容

右边灰色部分表示这一行被删除了

黄色部分表示有修改/添加



8. 解决冲突

修改好了提交之前，先做更新，如果遇到下面提示说明有冲突：

![1544434312593](C:\Users\yangyanqiu\AppData\Roaming\Typora\typora-user-images\1544434312593.png)

提交reamde文件的修改，遇到冲突，生成下面3个文件：

readme.md.mine：6版本内容+自己的修改

readme.md.r6：6版本readme的内容

readme.md.r9：readme现在最新版本9的内容

文件readme的内容会出现下面内容:

```
<<<<<<< filename
    你的修改
|||||| old revision
=======
    来自版本库中的代码
>>>>>>> new revision
```

有两种解决方法：

A、放弃自己的更新，使用svn revert（回滚），然后再重新修改提交。在这种方式下不需要使用svn resolved（解决）

B、手动解决：冲突发生时，通过和其他用户沟通之后，手动更新冲突文件。然后执TortoiseSVN->Resolved filename来解除冲突，最后提交。

更多使用说明参考文档：http://svndoc.iusesvn.com/tsvn/

### 3.2 在Ubuntu上使用

1. 查看svn命令帮助文档

   svn --help

2. 检出项目

   svn checkout 远程仓库路径 --usernmae 用户名 --password 用户密码

3. 更新项目

   svn update

4. 查看日志

   svn log

5. 提交修改

   svn commit -m "Commit message"

6. 撤销更改

   svn revert

7. 查看文件修改

   svn diff

8. 解决冲突