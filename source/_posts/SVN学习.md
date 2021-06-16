---
title: SVN学习
date: 2021-06-16 08:54:45
tags: [svn]
---

# SVN 简介

## SVN 的一些概念

- **repository（源代码库）:**源代码统一存放的地方
- **Checkout（提取）:**当你手上没有源代码的时候，你需要从repository checkout一份
- **Commit（提交）:**当你已经修改了代码，你就需要Commit到repository
- **Update (更新):**当你已经Checkout了一份源代码， Update一下你就可以和Repository上的源代码同步，你手上的代码就会有最新的变更

## SVN 的主要功能

- （1）目录版本控制

  CVS 只能跟踪单个文件的历史, 不过 Subversion 实作了一个 "虚拟" 的版本控管文件系统, 能够依时间跟踪整个目录的变动。 目录和文件都能进行版本控制。

- （2）真实的版本历史

  自从CVS限制了文件的版本记录，CVS并不支持那些可能发生在文件上，但会影响所在目录内容的操作，如同复制和重命名。除此之外，在CVS里你不能用拥有同样名字但是没有继承老版本历史或者根本没有关系的文件替换一个已经纳入系统的文件。在Subversion中，你可以增加（add）、删除（delete）、复制（copy）和重命名（rename），无论是文件还是目录。所有的新加的文件都从一个新的、干净的版本开始。

- （3）自动提交

  一个提交动作，不是全部更新到了档案库中，就是不完全更新。这允许开发人员以逻辑区间建立并提交变动，以防止当部分提交成功时出现的问题。

- （4）纳入版本控管的元数据

  每一个文件与目录都附有一組属性关键字并和属性值相关联。你可以创建, 并儲存任何你想要的Key/Value对。 属性是随着时间来作版本控管的,就像文件內容一样。

- （5）选择不同的网络层

  Subversion 有抽象的档案库存取概念, 可以让人很容易地实作新的网络机制。 Subversion 可以作为一个扩展模块嵌入到Apache HTTP 服务器中。这个为Subversion提供了非常先进的稳定性和协同工作能力，除此之外还提供了许多重要功能: 举例来说, 有身份认证, 授权, 在线压缩, 以及文件库浏览等等。还有一个轻量级的独立Subversion服务器， 使用的是自定义的通信协议, 可以很容易地通过 ssh 以 tunnel 方式使用。

- （6）一致的数据处理方式

  Subversion 使用二进制差异算法来异表示文件的差异, 它对文字(人类可理解的)与二进制文件(人类无法理解的) 两类的文件都一视同仁。 这两类的文件都同样地以压缩形式储存在档案库中, 而且文件差异是以两个方向在网络上传输的。

- （7）有效的分支(branch)与标签(tag)

  在分支与标签上的消耗并不必一定要与项目大小成正比。 Subversion 建立分支与标签的方法, 就只是复制该项目, 使用的方法就类似于硬连接（hard-link）。 所以这些操作只会花费很小, 而且是固定的时间。

- （8）Hackability

  Subversion没有任何的历史包袱; 它主要是一群共用的 C 程序库, 具有定义完善的API。这使得 Subversion 便于维护, 并且可被其它应用程序与程序语言使用。

## 优于CVS之处

​	1、原子提交。一次提交不管是单个还是多个文件，都是作为一个整体提交的。在这当中发生的意外例如传输中断，不会引起数据库的不完整和数据损坏。

​	2、重命名、复制、删除文件等动作都保存在版本历史记录当中。

​	3、对于二进制文件，使用了节省空间的保存方法。（简单的理解，就是只保存和上一版本不同之处）

​	4、目录也有版本历史。整个目录树可以被移动或者复制，操作很简单，而且能够保留全部版本记录。

​	5、分支的开销非常小。

​	6、优化过的数据库访问，使得一些操作不必访问数据库就可以做到。这样减少了很多不必要的和数据库主机之间的网络流量。

# SVN 安装

https://www.runoob.com/svn/svn-install.html

## SVN 生命周期

## 创建版本库

​	版本库相当于一个集中的空间，用于存放开发者所有的工作成果。版本库不仅能存放文件，还包括了每次修改的历史，即每个文件的变动历史。

Create 操作是用来创建一个新的版本库。大多数情况下这个操作只会执行一次。当你创建一个新的版本库的时候，你的版本控制系统会让你提供一些信息来标识版本库，例如创建的位置和版本库的名字。

## 检出

​	Checkout 操作是用来从版本库创建一个工作副本。工作副本是开发者私人的工作空间，可以进行内容的修改，然后提交到版本库中。

## 更新

​	顾名思义，update 操作是用来更新版本库的。这个操作将工作副本与版本库进行同步。由于版本库是由整个团队共用的，当其他人提交了他们的改动之后，你的工作副本就会过期。

让我们假设 Tom 和 Jerry 是一个项目的两个开发者。他们同时从版本库中检出了最新的版本并开始工作。此时，工作副本是与版本库完全同步的。然后，Jerry 很高效的完成了他的工作并提交了更改到版本库中。

​	此时 Tom 的工作副本就过期了。更新操作将会从版本库中拉取 Jerry 的最新改动并将 Tom 的工作副本进行更新。

## 执行变更

​	当检出之后，你就可以做很多操作来执行变更。编辑是最常用的操作。你可以编辑已存在的文件，例如进行文件的添加/删除操作。

​	你可以添加文件/目录。但是这些添加的文件目录不会立刻成为版本库的一部分，而是被添加进待变更列表中，直到执行了 commit 操作后才会成为版本库的一部分。

​	同样地你可以删除文件/目录。删除操作立刻将文件从工作副本中删除掉，但该文件的实际删除只是被添加到了待变更列表中，直到执行了 commit 操作后才会真正删除。

​	Rename 操作可以更改文件/目录的名字。"移动"操作用来将文件/目录从一处移动到版本库中的另一处。

### 复查变化

​	当你检出工作副本或者更新工作副本后，你的工作副本就跟版本库完全同步了。但是当你对工作副本进行一些修改之后，你的工作副本会比版本库要新。在 commit 操作之前复查下你的修改是一个很好的习惯。

​	Status 操作列出了工作副本中所进行的变动。正如我们之前提到的，你对工作副本的任何改动都会成为待变更列表的一部分。Status 操作就是用来查看这个待变更列表。

​	Status 操作只是提供了一个变动列表，但并不提供变动的详细信息。你可以用 diff 操作来查看这些变动的详细信息。

## 修复错误

​	我们来假设你对工作副本做了许多修改，但是现在你不想要这些修改了，这时候 revert 操作将会帮助你。

​	Revert 操作重置了对工作副本的修改。它可以重置一个或多个文件/目录。当然它也可以重置整个工作副本。在这种情况下，revert 操作将会销毁待变更列表并将工作副本恢复到原始状态。

## 解决冲突

​	合并的时候可能会发生冲突。Merge 操作会自动处理可以安全合并的东西。其它的会被当做冲突。例如，"hello.c" 文件在一个分支上被修改，在另一个分支上被删除了。这种情况就需要人为处理。Resolve 操作就是用来帮助用户找出冲突并告诉版本库如何处理这些冲突。

## 提交更改

​	Commit 操作是用来将更改从工作副本到版本库。这个操作会修改版本库的内容，其它开发者可以通过更新他们的工作副本来查看这些修改。

​	在提交之前，你必须将文件/目录添加到待变更列表中。列表中记录了将会被提交的改动。	当提交的时候，我们通常会提供一个注释来说明为什么会进行这些改动。这个注释也会成为版本库历史记录的一部分。Commit 是一个原子操作，也就是说要么完全提交成功，要么失败回滚。用户不会看到成功提交一半的情况。

# SVN 启动模式

```sh
yum install -y subversion
```

https://www.runoob.com/svn/svn-start-mode.html

# SVN 创建版本库

```sh
[runoob@centos6 ~]# svnadmin create /opt/svn/runoob01
[runoob@centos6 ~]# ll /opt/svn/runoob01/
total 24
drwxr-xr-x 2 root root 4096 2016/08/23 16:31:06 conf
drwxr-sr-x 6 root root 4096 2016/08/23 16:31:06 db
-r--r--r-- 1 root root    2 2016/08/23 16:31:06 format
drwxr-xr-x 2 root root 4096 2016/08/23 16:31:06 hooks
drwxr-xr-x 2 root root 4096 2016/08/23 16:31:06 locks
-rw-r--r-- 1 root root  229 2016/08/23 16:31:06 README.txt
```

​	进入/opt/svn/runoob01/conf目录 修改默认配置文件配置，包括svnserve.conf、passwd、authz 配置相关用户和权限。

**svn服务配置文件svnserve.conf**

​	svn服务配置文件为版本库目录中的文件conf/svnserve.conf。该文件仅由一个[general]配置段组成。

```
[general]
anon-access = none
auth-access = write
password-db = /home/svn/passwd
authz-db = /home/svn/authz
realm = tiku 
```

- **anon-access:** 控制非鉴权用户访问版本库的权限，取值范围为"write"、"read"和"none"。 即"write"为可读可写，"read"为只读，"none"表示无访问权限。 默认值：read
- **auth-access:** 控制鉴权用户访问版本库的权限。取值范围为"write"、"read"和"none"。 即"write"为可读可写，"read"为只读，"none"表示无访问权限。 默认值：write
- **authz-db:** 指定权限配置文件名，通过该文件可以实现以路径为基础的访问控制。 除非指定绝对路径，否则文件位置为相对conf目录的相对路径。 默认值：authz
- **realm:** 指定版本库的认证域，即在登录时提示的认证域名称。若两个版本库的 认证域相同，建议使用相同的用户名口令数据文件。 默认值：一个UUID(Universal Unique IDentifier，全局唯一标示)。

**用户名口令文件passwd**

​	用户名口令文件由svnserve.conf的配置项password-db指定，默认为conf目录中的passwd。该文件仅由一个[users]配置段组成。

[users]配置段的配置行格式如下：

```
<用户名> = <口令>
```

```
[users]
admin = admin
thinker = 123456
```

**权限配置文件**

​	权限配置文件由svnserve.conf的配置项authz-db指定，默认为conf目录中的authz。该配置文件由一个[groups]配置段和若干个版本库路径权限段组成。

​	[groups]配置段中配置行格式如下：

```
<用户组> = <用户列表>
```

​	版本库路径权限段的段名格式如下:

```
[<版本库名>:<路径>] 
```

```
[groups]
g_admin = admin,thinker

[admintools:/]
@g_admin = rw
* =

[test:/home/thinker]
thinker = rw
* = r
```

# SVN 检出操作

​	上一章中，我们创建了版本库runoob01,URL为svn://192.168.0.1/runoob01,svn用户user01有读写权限。

​	我们就可以通过这个URL在客户端对版本库进行检出操作。

​	svn checkout http://svn.server.com/svn/project_repo --username=user01 以上命令将产生如下结果：

```sh
root@runoob:~/svn# svn checkout svn://192.168.0.1/runoob01 --username=user01
A    runoob01/trunk
A    runoob01/branches
A    runoob01/tags
Checked out revision 1.
```

​	检出成功后在当前目录下生成runoob01副本目录。查看检出的内容

```sh
root@runoob:~/svn# ll runoob01/
total 24
drwxr-xr-x 6 root root 4096 Jul 21 19:19 ./
drwxr-xr-x 3 root root 4096 Jul 21 19:10 ../
drwxr-xr-x 2 root root 4096 Jul 21 19:19 branches/
drwxr-xr-x 4 root root 4096 Jul 21 19:19 .svn/
drwxr-xr-x 2 root root 4096 Jul 21 19:19 tags/
drwxr-xr-x 2 root root 4096 Jul 21 19:19 trunk/
```

# SVN 解决冲突

​	假设 A、B 两个用户都在版本号为 100 的时候，更新了 kingtuns.txt 这个文件，A 用户在修改完成之后提交 kingtuns.txt 到服务器， 这个时候提交成功，这个时候 kingtuns.txt 文件的版本号已经变成 101 了。同时B用户在版本号为 100 的 kingtuns.txt 文件上作修改， 修改完成之后提交到服务器时，由于不是在当前最新的 101 版本上作的修改，所以导致提交失败。

我们已在本地检出 runoob01 库，下面我们将实现版本冲突的解决方法。

我们发现 HelloWorld.html 文件存在错误，需要修改文件并提交到版本库中。

我们将 HelloWorld.html 的内容修改为 "HelloWorld! https://www.runoob.com/"。

```sh
root@runoob:~/svn/runoob01/trunk# cat HelloWorld.html 
HelloWorld! http://www.runoob.com/
```

用下面的命令查看更改：

```sh
root@runoob:~/svn/runoob01/trunk# svn diff 
Index: HelloWorld.html
===================================================================
--- HelloWorld.html     (revision 5)
+++ HelloWorld.html     (working copy)
@@ -1,2 +1 @@
-HelloWorld! http://www.runoob.com/
+HelloWorld! http://www.runoob.com/!
```

尝试使用下面的命令来提交他的更改：

```sh
root@runoob:~/svn/runoob01/trunk# svn commit -m "change HelloWorld.html first"
Sending        HelloWorld.html
Transmitting file data .svn: E160028: Commit failed (details follow):
svn: E160028: File '/trunk/HelloWorld.html' is out of date
```

这时我发现提交失败了。

​	因为此时，HelloWorld.html 已经被 user02 修改并提交到了仓库。Subversion 不会允许 user01(本例使用的 svn 账号)提交更改，因为 user02 已经修改了仓库，所以我们的工作副本已经失效。

​	为了避免两人的代码被互相覆盖，Subversion 不允许我们进行这样的操作。所以我们在提交更改之前必须先更新工作副本。所以使用 update 命令，如下：

```sh
root@runoob:~/svn/runoob01/trunk# svn update
Updating '.':
C    HelloWorld.html
Updated to revision 6.
Conflict discovered in file 'HelloWorld.html'.
Select: (p) postpone, (df) show diff, (e) edit file, (m) merge,
        (mc) my side of conflict, (tc) their side of conflict,
        (s) show all options: mc
Resolved conflicted state of 'HelloWorld.html'
Summary of conflicts:
  Text conflicts: 0 remaining (and 1 already resolved)
```

​	这边输入"mc",以本地的文件为主。你也可以使用其选项对冲突的文件进行不同的操作。

​	默认是更新到最新的版本，我们也可以指定更新到哪个版本

​	此时工作副本是和仓库已经同步，可以安全地提交更改了

```sh
root@runoob:~/svn/runoob01/trunk# svn commit -m "change HelloWorld.html second"
Sending        HelloWorld.html
Transmitting file data .
Committed revision 7.
```

# SVN 提交操作

​	在上一章中，我们检出了版本库runoob01，对应的目录放在/home/user01/runoob01中，下面我们针对这个库进行版本控制。

------

​	我们在库本版中需要增加一个readme的说明文件。

```sh
root@runoob:~/svn/runoob01/trunk# cat readme 
this is SVN tutorial.
```

​	查看工作副本中的状态。

```sh
root@runoob:~/svn/runoob01/trunk# svn status
?       readme
```

​	此时 readme的状态为？，说明它还未加到版本控制中。

​	将文件readme加到版本控制，等待提交到版本库。

```sh
root@runoob:~/svn/runoob01/trunk# svn add readme 
A         readme
```

​	查看工作副本中的状态

