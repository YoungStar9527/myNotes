

# 1 git背景

**1 本地版本控制系统**

​	最早的版本控制系统了，比较知名的是**RCS**，就是在本地对你的文件的每次修改维护一系列的patch，每个patch就是两个文件版本之间的修改。然后如果你要回到某个历史版本上，RCS会通过path的应用来恢复任何一个时间点的文件。

​	这个方式比较Low，常见于上世纪八十年代，就一个程序员写一个系统，编程，只要自己可以管理自己的代码版本即可，不需要跟别人协作。

**2 集中式版本控制系统**

​	后来人们遇到的问题就是，不是只有自己需要在本地对文件版本进行控制，尤其是软件开发人员，需要多人写作，对同一份代码进行版本控制。比如每个人都要对代码进行修改，那么每个人都会更新出一个版本的代码。

​	这个时候就需要集中式的版本控制系统，就是弄一个服务器，上面放所有代码文件，并且进行版本控制，接着多个开发人员在本地连接服务器，进行代码从服务器检出到本地，代码的修改，以及代码修改的提交到服务器。

​	比较知名的有**CVS**和**SVN**，很长一段时间内，尤其在上个世纪的90年代到2000年初期，基本就是版本控制的标准。持续到了2005年，2010年之前，还是蛮广泛的。SVN，比如说，大名鼎鼎的百度，当年也是SVN去做代码版本控制。

​	但是集中式版本控制系统最大的问题，就在于单点故障，如果服务器故障一段时间，那么那段时间里，各个开发人员几乎啥也干不了了，没法从服务器检出最新代码，没法提交代码。如果更加坑爹的事情发生，就是服务器的磁盘坏了，结果还没有备份，那么完蛋了，所有的代码全部丢失。

**3 分布式版本控制系统**

​	这种模式下，每个开发人员自己的本地电脑，都会从服务器检出一份完整的代码包括历史所有的版本到本地。相当于每个开发人员本地都有一份完整的代码版本的副本拷贝。此外，每个开发人员自己本地都有一个完整的版本控制系统。

​	如果服务器有任何故障，开发人员在自己本地可以做所有的工作，包括代码版本的切换，代码修改的提交，等等所有的版本控制操作，不会影响自己的工作。如果服务器磁盘坏了，数据丢失，那么可以将某个开发人员本地的版本库拷贝一份到服务器，去恢复数据即可。

大名鼎鼎的分布式版本控制系统，就是**Git**

**优点：**

​	(1) 服务器所有代码在本地都有副本拷贝

​	(2) 服务器断网故障，本地也可以使用，不需要联网能进行版本控制

​	(3) 服务器磁盘损坏，代码全都丢失，也可以基于本地代码进行恢复

# 2 git绑定远程仓库

## 2.1 下载git，安装git，一直下一步即可，傻瓜式操作

## 2.2绑定github

```shell
ssh-keygen -t rsa -C "your_email@youremail.com"
```

后面的是注册GitHub时候的邮箱地址，后面的一些操作我们默认回车就可以

![image-20210530102316176](git基本操作.assets/image-20210530102316176.png)

![image-20210530102328489](git基本操作.assets/image-20210530102328489.png)

然后绑定SSH key，进入github

![image-20210530102522719](git基本操作.assets/image-20210530102522719.png)

其中的title随便填，下面的粘贴在你电脑上生成的key。点击添加之后，则添加成功：

![image-20210530102553923](git基本操作.assets/image-20210530102553923.png)



验证是否绑定本地成功，在git-bash中验证，输入指令：

```shell
ssh -T git@github.com
```

![image-20210530102816418](git基本操作.assets/image-20210530102816418.png)

由于github每次执行commit操作时，都会记录username和email，所以需要设置

```shell
git config --global user.name "youngstar"
git config --global user.email "youngstar@123.com"
```

## 2.3 本地git目录绑定，上传github

### 2.3.1 建立远程仓库

![image-20210530103403458](git基本操作.assets/image-20210530103403458.png)

输入远程仓库名,及打勾README file文件，初始仓库需要一个默认文件，才能绑定

![image-20210530103459406](git基本操作.assets/image-20210530103459406.png)

### 2.3.2 本地仓库绑定远程仓库

本地目录也新建一个空的README.MD

![image-20210530103646057](git基本操作.assets/image-20210530103646057.png)

在本地需要上传/绑定github目录下输入git初始化命令

```shell
git init
```

然后将目录下文件提交到本地仓库

```shell
git add.
git commit -m "first commit"
```

复制远程仓库ssh

![image-20210530103854256](git基本操作.assets/image-20210530103854256.png)

然后关联远程仓库，并推送到远程仓库

```shell
git remote add origin git@github.com:wangjiax9/practice.git 
#关联远程仓库，用ssh关联
git push -u origin master 
#把本地库的所有内容推送到远程库上
```

### 2.3.3 修改github默认展示分支

可以将master设为默认展示分支

![image-20210530104036222](git基本操作.assets/image-20210530104036222.png)

## 2.4 linux建立远程仓库并绑定

安装git

```shell
git --version
#先查看没有就直接安装
yum install -y git
```

在centos上创建一个git用户来运行git服务：

```shell
groupadd git
adduser git -g git
```

**windows本地生成对应秘钥，重命名rsa避免覆盖之前的，输入生成秘钥命令根据提示取名**

```shell
ssh-keygen -t rsa
#生成秘钥在用户目录的.ssh目录下
#可选后续参数写邮箱
```

将公钥放入authorized_keys，如果没有该文件则创建

```shell
cd /home/git/
mkdir .ssh
chmod 755 .ssh
touch .ssh/authorized_keys
chmod 644 .ssh/authorized_keys
#把公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。
```

初始化Git仓库

使用/srv/oa-parent.git目录作为远程仓库，在/srv目录下输入命令：

```shell
git init --bare oa-parent.git
#Git会创建一个裸仓库，裸仓库没有工作区。因为服务器上的Git仓库就是为了共享，不会让用户直接登录到服务器上去修改工作区的，而且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：
chown -R git:git oa-parent.git
#将对应账户改为git
```

禁用shell登录

一般出于安全考虑，第二步创建的git用户是不允许使用shell的，我们可以编辑/etc/passwd文件

```shell
#找到类似下面的一行：
git:x:1001:1001:,,,:/home/git:/bin/bash
#改为：
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
#这样，git用户可以正常通过ssh使用git推送和下载代码，但是无法通过shell登录的，因为我们为git用户指定的git-shell是每次一登录就自动退出
```

基于git服务器和远程仓库上传和下载代码

```shell
git remote add origin ssh://git@192.168.31.244:/srv/oa-parent.git
#本地仓库，添加远程仓库

git push -u origin master
#将本地仓库和git服务器上的远程仓库关联起来
#-u，将本地仓库的master分支和远程仓库的master分支关联起来
```

在另一个文件夹克隆远程仓库(测试是否生效，如果克隆成功，代码和刚才上传的本地仓库一致则说明配置成功)

```shell
git clone ssh://git@192.168.31.244:/srv/oa-parent.git
```

大概流程

（1）搭建起来了一个git服务器

（2）在git服务器上创建了一个远程仓库，bare裸仓库

（3）将本地的项目代码推送到了远程仓库

（4）在本地另外一个目录，模拟成另外一个研发人员，在自己本地将远程仓库中的项目代码克隆到了本地

引用：https://www.cnblogs.com/HuSay/p/9101130.html

## 2.5 git配置多账户

如果配置了全局账户需要先删除全局账户

```shell
git config --list
#查看配置
```

![image-20210617211409425](git基本操作.assets/image-20210617211409425.png)

使用以下命令删除全局配置

```shell
# 移除全局配置账户
git config --global --unset user.name
#查看全局用户名
git config --global user.name
# 移除全局配置邮箱
git config --global --unset user.email
# 查看全局邮箱
git config --global user.email
# 移除全局密码
git config --global --unset user.password
# 查看全局密码
git config --global user.password

```

**生成对应秘钥，重命名rsa避免覆盖之前的，输入生成秘钥命令根据提示取名**

```shell
ssh-keygen -t rsa
#生成秘钥在用户目录的.ssh目录下
#可选后续参数写邮箱
```

![image-20210617211825691](git基本操作.assets/image-20210617211825691.png)

**将rsa公钥加入对应git远程仓库中**

不同远程仓库添加的位置不同，自行查询(github见2.2)

**配置config文件**

```shell
# 配置user1 
Host github.com
HostName github.com
IdentityFile C:\\Users\\lingh\\.ssh\\id_rsa
PreferredAuthentications publickey
User user1

# 配置user2
Host localhost
HostName localhost
IdentityFile C:\\Users\\lingh\\.ssh\\id_rsa2
PreferredAuthentications publickey
User user2

#Host    　　主机别名
#HostName　　服务器真实地址
#IdentityFile　　私钥文件路径
#PreferredAuthentications　　认证方式
#User　　用户名
#Host主机别名 相当于git@Host:YoungStar9527/myNotes.git(git@github.com:YoungStar9527/myNotes.git )
```

终端测试ssh key是否生效

```shell
ssh -T git@user1.github.com
ssh -T git@user2.github.com
#如果git远程仓库安全策略配置了禁用ssh直列会抛出异常，不过不影响正常使用(clone push pull等操作)
```

为git各仓库单独配置邮箱

```shell
git config user.name "user1"
git config user.email "user1@email.com"

```

引用：https://www.pianshen.com/article/302848775/

# 3 git配置与基本命令

## 3.1 git基本配置操作与命令

​	打开git-bash，进入一个模拟了linux环境的命令行的界面，然后通过这里就可以去操作git，同时也可以认为git-bash是一个windows上的linux命令行模拟器，因为linux上的大部分命令都可以执行，包括ssh

```shell
git --version
#检查一下git的版本号
```

在win10的用户目录下的.gitconfig文件为git的基本配置文件(类似maven的全局配置文件setting.xml)

```shell
C:\Users\18146\.gitconfig
#18146为用户目录名
```

​	一般都不会直接修改git的配置文件来进行配置，一般都是执行git config命令，来修改我们的各个不同的范围的配置，比如全局的配置，或者是项目特殊的配置

​	配置用户名和邮箱，安装好git之后第一个事情，就是配置自己的用户名和邮箱，后面每次你提交代码的时候，都会带上你的个人信息

```shell
git config --global user.name "zhss"
git config --global user.email "zhss@163.com"
git config --global
#就是设置当前用户范围内的配置，对机器上的其他用户是无效的
git config --system
#就是对当前机器上所有用户都生效
git config
#就是对当前所在的git项目本身生效
```

 **查看信息**

```shell
git config --list
#查看所有的配置项
git config user.name
#查看某个配置项
git add --help
#git帮助文档
```

**初始化git(在这个项目的根目录下面执行)**

```shell
git init
#执行了git init命令之后，意思就是说，要让git托管这个项目的代码，后续这个项目中的所有代码的变动，都可以作为一个版本提交到git中，git会管理这个项目的每个版本的代码，同时后续我们就可以基于git对这个项目进行各种各样的版本控制的功能  

#git init命令执行之后，做的第一件事情，此时就会创建一个.git隐藏目录，这里包含了git的所有的文件，就是每个版本的代码都会存储在.git目录中。之前我们不是说，如果要对HelloWorld.java文件进行版本控制的话，实际上是要保留这个文件的多个版本的代码的，如果自己手工做，那么就是保存多个代码文件。

#.git目录中，实际上就是存储了你的每个代码文件的每个版本，每个版本就是一个独立的文件。历史版本都被存储在了.git目录中。
```

**进行代码托管/版本控制/提交代码**

```shell
git add --all .
#将该目录下所有代码加入暂存区
git commit -m 'initial project version'
#将暂存区的代码提交到repository(仓库中)
```

 **gitignore配置(配置忽略文件)**

确保.gitignore文件和.git文件夹在同级目录

```shell
#         # 此为注释 – 将被 Git 忽略
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外（排除）
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

在.gitignore中已经标明忽略的文件，当git push的时候还会出现在push的目录中，原因是这些文件因为在git中有缓存，这时候我们就应该先把本地缓存删除，然后再进行git的push。git清除本地缓存命令如下：

```shell
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

引用：https://www.cnblogs.com/xuanjiange/p/13458618.html

**git add扩展**

```shell
git add --all .
#git add . 就是将当前新增或者是修改过的文件，加入暂存区，但是加了--all之后，如果有文件被删除，也会将文件被删除的状态加入暂存区中
#一般工作中，都是用git add --all .
#但是也可以直接git add 某个文件，把指定的文件加入到暂存区中
```



## 3.2 git解决中文展示乱码

1.打开git bash后，

```text
对窗口右键->Options->Text->Locale改为zh_CN，Character set改为UTF-8
关闭git bash，再打开，可以显示中文了。
```

2.如果前一种方法不行

则在git bash中尝试执行下面内容

```text
git config --global core.quotepath false  
关闭git bash，再打开，可以显示中文了。
```

3.如果前面不行

```text
git config --global i18n.commitencoding utf-8    #如果是GBK 请换成gbk
git config --global i18n.logoutputencoding utf-8   #如果是GBK 请换成gbk
export LESSCHARSET=utf-8
关闭git bash，再打开，可以显示中文了。
```

引用：https://zhuanlan.zhihu.com/p/114362960

## 3.3 基于git log查看提交历史

在我们提交了很多次之后，可以查看提交历史，使用git log命令即可，可以看到每次commit的信息，包括了commit的SHA-1、作者、日期、提交说明。

![image-20210616220918733](git基本操作.assets/image-20210616220918733.png)

**HEAD -> master的含义**

![image-20210616221012741](git基本操作.assets/image-20210616221012741.png)

HEAD，指针，指向了我们当前所处的分支，因为当前我们默认就处于master分支上，所以HEAD指针就是指向master的

**git log常用参数**

```shell
git log --patch -2，
#--patch可以显示每次提交之间的diff，同时-n可以指定显示最近几个commit。这个是很有用的，可以看最近两次commit之间的代码差异，进行code review是比较方便的。
 
git log --stat
#可以显示每次commit的统计信息，包括修改了几个文件，有多少行插入，多少行删除。
 
git log --pretty=oneline
#可以每个commit显示一行，就是一个commit SHA-1和一个提交说明。用git log --pretty=format:"%h - %an, %ar : %s"，可以显示短hash、作者、多长时间以前、提交说明。
 
git log --oneline --abbrev-commit --graph
#这是最有用的，可以看到整个commit树结构，包括如何合并的，就显示每个commit的SHA-1和提交说明，同时SHA-1显示短值。

#--oneline：显示一行，不要显示多行那么多东西，一行里，就显示commit的标识符，SHA-1 hash值，40位的；提交备注；显示分支和HEAD指向哪个commit

#--abbrev-commit：commit的标识符，每一次commit，都有一个唯一的标识符，就是一个SHA-1 hash值，40位，显示一个短值，默认显示前7位，就是说前7位就可以唯一定位这个commit了，不需要完整的40位

#--graph：显示图形化的commit历史，这个大家后面学习到分支那里就知道了，如果有分支的话，commit历史会形成一棵树的形状，这个时候用--graph可以看清楚这颗commit树长什么样子，很有的

```

**git提交历史深入剖析图**

![image-20210616221320799](git基本操作.assets/image-20210616221320799.png)

**图中名词解释(下图)：blob就是版本，tree object就是指针指向对应的版本文件，commit object就是对应版本提交(git log中对应的信息)，comment就是提交代码的注释**

**图中流程解释：下图对应的是上图commit(提交)到仓库后的版本对应信息，master指向的是最新的版本代码，最新版本代码又保留上一个的版本的指针**



​	在工作区修改代码之后，执行git add命令，会将代码放入版本库中的暂存区；接着执行git commit命令之后，会将暂存区中的代码提交到版本库中的master分支上，而HEAD指针就指向master分支的最新一次提交。

​	所以现在我们就很清楚了，git add，其实就是可以多次修改代码，多次git add，然后将每次修改的代码，都放入暂存区中；而git commit，就是一次性将暂存区中的代码，全部提交到master分支上，master分支会出现一个最新的commit，也就是一个最新的代码版本；而HEAD作为一个指针，永远指向master分支的最新一次commit的代码版本。

## 3.4 git版本控制

```shell
git reset --hard HEAD^
#就可以回退到上一个版本
#HEAD^，代表了什么？
#HEAD -> master -> commit（add methods for classes）
#HEAD^，代表的是commit（add methods for classes）的上一个commit（add two files）
git reset --hard HEAD^
#就是将仓库、暂存区、工作区，全部恢复到上一个commit（add two files）对应的状态
git reset --hard HEAD~5
#退回到HEAD之前的倒数第5个commit的状态
git reset --hard d324644
#指定一个commit的hash值，回退到很老的版本(一般使用hash值得前7位即可)
#在不引起歧义的情况下，可以用前面n位。git命令中支持的最短的hash值是4位的
#于“歧义”的例子，假如某个仓库里有个object的名字/hash值前五位是12345，另一个前五位是12346，那么用1234就有歧义
git reset
#这个命令，可以任意穿梭到历史的任何一个版本上去
#如果我要回来呢？重新回到之前add methods for classes那个commit对应的版本
git reflog
#查看git reflog 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作）
git show master
#可以查看master分支指向的那个commit
git show HEAD^
git show HEAD~2
#HEAD，指针，指向的是当前的那个分支，当前的那个分支又是指向某个commit
#HEAD，就可以指代当前你所处的那个commit对应的版本
#HEAD^，就是说，HEAD对应的commit的上一个commit，HEAD^^，上两个commit，HEAD^^^，上三个commit
#HEAD~5，就是说HEAD之前的第5个commit
```

**PS:我们不一定就是要用40位完整的SHA-1 hash值来选择一个commit，也可以只是提供hash值的前几位就可以了，至少要4位以上，同时在git仓库中就这前几位可以唯一定位一个commit就ok.就是说，我们如果提供hash值的前7位，git会自动唯一定位出来这个前7位对应的是哪个commit**

**git log和git reflog的区别：**

​	**git log：**是你当前的分支指向的commit之前的所有commit会显示出来，你的分支指向的那个commit之后的commit是不会显示的

​	**git reflog：**是会显示最近几个月，完整的一个commit历史的（reflog是每个研发人员自己本地的电脑上的记录，跟git log显示出来的提交历史不同，不会同步到远程仓库上去的）

**git最核心的原理**

（1）三个区域（三棵树）：工作区、暂存区、仓库

（2）提交历史：在仓库中，blob->tree->commit->master-HEAD，这套东西形成了git的数据结构

（3）commit树+分支指针：无论在哪个分支上做开发和提交，都是在这个项目公共的一个commit树上在添加不同的commit，形成一颗越来越长的树。分支只是指向某个commit的一个指针而已。然后每个commit就代表了那个时刻下，项目的一个完整的历史快照版本。

（4）因为分支实际上就是一些指针，所以分支是指向commit的，也可以直接用分支名称来指代它目前指向的那个commit

![image-20210617214833936](git基本操作.assets/image-20210617214833936.png)

# 4 git本地仓库结构与文件状态

## 4.1 git本地仓库结构

git项目有3个主要的部分组成：

工作区（working directory / working tree），暂存区（staging area），版本库（git directory / repository）

**working directory / working tree：工作区，**保存的是一个项目当前的一个版本对应的所有文件，这些文件是从git版本库中的压缩后的数据库中提取出来，然后放到我们的磁盘上去。

**staging area：暂存区**，就是一个文件，包含在git版本库中，主要是保存了下一次要提交到的那些文件信息。在git中，对暂存区有另外一个名称，叫做index，也就是索引。

**git directory / repository：git版本库**，其实就是git用于存储自己的元数据，以及文档数据库的地方，默认就是在项目的.git隐藏目录中

上面三个区域的协作关系大致如下：

（1）首先会在工作区修改某个版本的文件

（2）将某些修改后的文件放入git暂存区中，准备下一次提交到git版本库中去

（3）执行一个提交操作，将暂存区中的文件保作为一个快照保存到git版本库中去

如果一个文件，已经有一个版本被保存到了版本库，那么就是committed状态；如果这个文件被修改了，同时被加入了暂存区，那么就是staged状态；如果这个文件修改了，还没有加入暂存区，那么就是modified状态。

工作区，working directory

所谓的工作区，指的就是当前你的git管理的项目，在本地的那个目录，也就是你能直接看到，编辑的那个目录，这就是工作区。

![image-20210616211948521](git基本操作.assets/image-20210616211948521.png)

## 4.2 git相关机制

（1）**快照机制**：每次提交文件，都是保存一份这个文件当前这个状态的一个完整快照，同时对这次提交维护一个指针，指向这个文件快照(而CVS，SVN等，是通过一开始提交一个原始文件，然后后面每次对文件进行修改之后再次提交，都维护这次提交对应的一个差异)

（2）**本地化操作**：大多数的git版本控制操作，只要在本地执行即可，所有的版本文件都在本地，因此操作是非常快速的。比如说通过git查看提交历史，比较历史文件的差异，都可以在本地完成，不需要通过服务器做任何事情

（3）**完整性保证**：git在存储任何文件之前，都会对其执行一个校验和，然后用校验和指向那个文件。

​	这是git内核保证的，这样我们是不可以手工修改git版本库中的任何文件的，因为修改了文件之后，会导致计算出来的校验和与之前保存的校验和不匹配，文件会破损。

​	git用的是**SHA-1** hash算法来计算校验和，这是一个40位的字符串，基于文件的内容计算出来的，看起来大概是这样的：

​	24b9da6552252987aa493b52f8696cd6d3b00373

​	如果手动破坏.git中存储的文件的内容，git会不承认，因为内容变化之后，会导致内容计算出来的SHA-1 40位的hash值变化，跟之前存储的hash值不同，就认为文件破损

（4）**仅仅添加数据**：基本是向仓库(repository)添加数据，基本不会丢失。

## 4.2 git文件状态

**通过git status查看文件状态**

![image-20210616213342096](git基本操作.assets/image-20210616213342096.png)

**PS:git默认中文展示会乱码(下图是配置展示中文设置后)**

![image-20210616214949829](git基本操作.assets/image-20210616214949829.png)

**PS:绿色就是已经add(暂存区/staged),红色就是没有add(工作区)**

**代码文件分成两种，一种是tracked，一种是untracked**。tracked文件就是已经提交到git版本库中的文件，后面可以处于modified或者staged状态；untracked文件，就是从来没有提交到git版本库的代码文件（也从来没有放入暂存区）。

**git管理的文件有三种状态：提交状态（committed），修改状态（modified），暂存状态（staged）。**

提交状态：我们的文件已经安全的保存在git的本地数据库中了。

修改状态：我们修改了文件，但是还没有提交到git的数据库中去。

暂存状态：将修改后的文件标记为即将通过下一次提交，保存到git数据库中去。

（1）新文件刚创建：untracked，此时仅仅停留在工作区中

（2）git add 新文件：new file，此时已经被追踪了，放入了暂存区中 => staged

（3）git commit 新文件：committed，已经被追踪了，放入了git仓库中 => committed

（4）修改那个文件：modified，changes not staged to be committed，没有加入暂存区，被修改的内容仅仅停留在工作区中 => modified

（5）git add 修改文件：modified，changes to be committed，修改的文件版本被已经加入暂存区 => staged

（6）git commit 修改文件：committed，修改后的新版本提交到了git仓库中 => committed

​	一般自己创建的版本库，刚开始文件都是untracked，然后git add和git commit命令执行之后，就被提交了第一个版本到git版本库，此时就全部都是tracked了。如果是从远程版本库克隆下来的，那么刚开始就是tracked。

​	接着对tracked的文件修改之后，就是modified；然后对modified文件再执行git add命令之后，就是staged，进入了暂存区；接着执行git commit命令之后，就将暂存区中的文件都提交到了版本库中，此时就是unmodified，and tracked。

**PS:在对文件修改没有add的情况对文件来说就是modified状态在回到工作区，add后才是modfied状态在暂存区。**

**PS:必须git add到暂存区里的修改，才会被git commit时提交到仓库里，否则就是停留在工作区中**

# 5 深度解析分支原理

## 5.1 基于git远程仓库的多人协作开发

![image-20210617215159324](git基本操作.assets/image-20210617215159324.png)





## 5.2 分支的基础知识

### 5.2.1 分支的介绍

​	所谓的分支功能，就是可以同时拉出来多个代码副本，然后在不同的代码副本上，可以进行对应功能的开发。完成开发之后，可以将多个分支合并在一起，形成最终的代码。

​	在git中，每一个项目，不管你有多少个分支，不管你在哪个分支上开发

​	最终都会形成一个完整的提交历史，树形结构

​	**每个分支，其实就只是一个指针而已，分支就指向了提交历史中的某个commit object**

​	**每个commit object就代表了这个项目的所有代码在那次提交的时候一个完整的快照版本**，包含了之前没有变更的代码文件，也包括了这次提交的最新的修改/新增/删除的代码文件

​	Git的分支功能是一个很大的特色，非常的轻量级，分支的来回切换速度是很快的，在实际的开发过程中，分支绝对是最重要的功能。

### 5.2.2 commit object工作原理

​	Git并不是存储一系列的文件差异，而是存储一系列的文件快照的。实际上每次我们执行一次commit，git都会存储一个commit object，这个commit object中会包含一个指针，指向这次提交文件的快照。这个commit object同时也包含作者的姓名和邮箱，提交说明，以及对上一次commit object的指针。

​	将一个文件版本放入暂存区的时候，就会计算一个校验和，然后提交的时候会将文件内容以blob的方式放入版本库中，同时在暂存区放入这个文件版本的校验和。接着git会创建一个commit object，其中会包含元数据，以及一个指针指向版本库中的文件快照。

​	也就是说，**每次执行一次提交，都会在版本库中包含这么几个东西：一个blob，每个文件都会有一个blob来存储这个文件的本次提交的快照；一个tree，这个tree会包含对本次提交的所有文件的blob的指针；一个commt object，指向了tree的指针，作者，等信息。**

​	接着如果再次执行一个提交，那**么下一个提交同样会包含那些东西**：每个文件一个blob，一个tree指向所有blob，一个commit object指向那个tree，同时这个commit object会有一个指针**，指向上一个commit object**。

​	最后多次提交，就会得到一颗完整的commit树。

​	分支是啥？分支就是一个轻量级的指针，默认的分支是master，那么每次提交，master分支的指针默认就是指向最新的那个commit object的。每次提交一次，master指针就会挪动，继续指向最新的commit object。

​	**git维护了一个特殊的指针HEAD，默认指向master分支指针，当本地切换到其他分支时，HEAD指针就会指向那个分支。HEAD指针就是当前工作区域的指针(当前指针，并非最新指针，当前指针HEAD可以指向之前的commit object)。**



### 5.2.3 创建一个分支

```shell
git branch testing
#可以创建一个新的分支，此时这个分支的指针会指向当前你所在的分支所指向的commit object上。
git log --oneline --decorate
#查看各个分支指向哪个commit object
```



### 5.2.4 切换分支

```shell
git checkout testing
#此时就可以切换到testing分支，此时HEAD指针会指向testing分支指针。
```

​	此时如果在testing分支上提交代码，那么会commit树会长出来一个新的commit object，而testing分支指正会指向最新的commit object，HEAD继续指向testing分支指针，而master指针还是指向之前的那个commit object。

​	git checkout master，会切换回master分支，此时HEAD指针会指向master指针，同时将master指针指向的那个commit object，对应的tree和其中的blob，也就是对应的文件快照恢复到工作区中来。

​	再次在master分支上提交一个修改，此时从这个commit object会再长出来一个新的commit object，看起来就是跟testing分支当前指向的commit object形成了两个分叉。

​	此时如果要查看commit树，可以用命令：

```shell
git log --oneline --decorate --graph --all
#可以打印出整颗commit树，同时告诉你各个分支当前指向哪个commit object
```

​	在Git中，**分支上就是一个很简单的文件，其中包含了一个40位的SHA-1校验和，就是分支指向的那个commit object的SHA-1。所以创建分支的代价是很低的，不过就是创建这么一个文件**罢了。而一些集中式版本控制系统，对于分支可能需要拷贝一个完整的文件副本，导致速度很慢，磁盘空间占用很大。

### 5.2.5  远程分支

**分支的提交与更新**

```shell
	git push -u origin 分支名称
	#将本地分支推送到远程仓库，默认分支名称一般为master
	#第一次推送且远程仓库没该分支，就是将本地分支推送到远程仓库里，形成一个同名的远程分支
	#-u这个选项就是将本地分支和远程分支关联起来 
	git push origin 分支名称
	#直接可以将本地分支的代码推送到远程分支(本地分支已和远程分支关联了)
	#此时你本地的commit树也会推送到远程版本库，跟远程版本库的commit树进行合并
	git pull
	#将当前分支在远程仓库的代码拉取下来，跟本地分支的代码进行合并
	#只会更新当前分支,其他分支新增的代码并不会合并
```

​	远程版本库的分支，在本地都有追踪分支(remote-tracking 分支)

​	比如说**本地的master分支，对应的远程分支就是origin/master。比如本地的feature/iss53分支，对应的远程分支就是origin/feature/iss53**

​	假设我们公司的git服务器地址是git.zhss.com，然后如果我们用git clone命令从这个git服务器克隆了一个版本库下来，git默认会将远程版本库命名为origin，同时在本地创建一个指向远程版本库的master分支的本地分支，叫做origin/master。此外，git也会给你在本地创建一个master分支，内容就是跟origin/master分支一样的。

​	从git服务器克隆版本库下来的时候，远程版本库的commit树会一同拷贝下来，然后origin/master指向的commit，就是远程版本库的master指向的commit，同时给本地创建的commit也是指向这个commit。

​	此时，如果在本地你做了不少开发，然后本地master移动了好几个commit，同时origin/master还是指向最开始的那个commit；而同时，远程版本库上，其他同事也提交了几次代码，因此远程版本库上的commit也移动了几个commit。

**拉取远程仓库分支并进行关联**

```shell
git fetch origin
#要让本地和远程保持同步
#同时拉取下来所有最新的远程分支
#本地的origin/master会指向远程版本库的master指向的那个commit，本地的master继续指向之前本地最新的那个commit
git checkout -b 分支名称 origin/分支名称
#可以将本地获取到一个分支跟origin/分支名称 关联起来，然后就可以跟你一起对一个分支进行修改了(因为git fetch origin获取到新分支时，这个新分支是只读的，需要关联起来，就可以对这个分支进行修改了)
#创建出来的本地分支叫做tracking brach，而这个本地分支track的远程分支叫做upstream branch。此时本地分支就会track那个远程分支，跟远程分支之间会建立关联关系

```

**分支合并与创建分支本地分支**

```shell
git merge 分支名称
#将分支代码合并到当前所在分支上去
#先git fetch origin，然后git merge 远程分支(如果冲突的话，可以一个一个解决)，相当于git pull(可能会冲突，形成冲突文件，是可读的)
git checkout -b 分支名称
#这个命令就是创建本地分支，本地分支指针指向最新一个commit，同时HEAD指针指向本地分支指针
#git checkout -b，相当于两个命令，git branch dev，git checkout dev，先创建分支，再切换到分支
```

​	如果我们在tracking分支上，执行git pull命令，git就会自动将对应的远程分支在远程版本库上的代码拉取下来跟本地的tracking分支做合并，如果有冲突的话，还需要解决冲突。

​	如果我们是使用clone命令克隆的远程版本库，那么默认就会将本地的master分支创建为追踪origin/master远程分支的。

**分支的版本控制**

```shell
git branch
#可以查看当前的所有分支以及我们目前所处的分支
git branch -u origin/分支名称
#可以让当前本地分支track(追踪)某个指定的远程分支
git branch -v
#显示出每个分支当前指向的commit object
git branch -vv
#查看每个本地分支track的远程分支
git branch
#显示出当前所有分支列表，以及你在哪个分支上工作
git branch --merged
#可以看到哪些分支被merge进了当前分支
git branch --no-merged
#可以看到哪些分支还没有被merge进当前分支
git branch -d 分支名称
#命令删除一个本地分支
#如果你有一个分支，还没有合并到master分支去，此时git不让你删除的，可以使用 git branch -D
git branch -D 分支名称
#强制删除一个没有合并到别的分支的分支
#git branch -d可能会提示你那个分支还没merge到当前分支来，不让你删除该分支，此时可以使用git branch -D命令，强制删除一个分支。
git push origin --delete 分支名称
#删除远程分支
```

### 5.2.6 分支知识总结

#### 5.2.6.1 基础知识总结

​	1 git分支的一些基础知识：commit提交历史，分支就是指针，HEAD指向当前分支

​	2 拉不同的分支，就是不同的指针，基于指针指向的那个版本的代码，你在分支上做开发提交，其实就是在commit提交树上长出来不同的枝丫

​	3 切换分支，就是切换HEAD指向的分支指针

​	4 分支合并，其实就是将两个分支当前指向的两个commit的代码，合并到一起，形成额一个新的commit

​	5 origin/master对应的就是远程仓库的master

​	6 本地仓库和远程仓库之间推送和拉取代码，实际上就是在同步整套commit提交历史，包括进行一些分支指针的合并

PS:不同分支查看git log只是查看当前分支的log

#### 5.2.6.2 知识结合实战总结

​	1、不管是谁，不管在哪里，不管是哪个分支，你做提交，其实就是在自己本地的提交历史树上加一个新的commit，然后你当前的分支就指向最新的那个commit而已

​	2、你切换分支，新建分支，只不过是在切换指向的commit而已；因为不同的分支是指向不同的commit的，对应的是不同版本的代码；新建分支只不过是多一个指针而已，指向当前的这个commit

​	3、在本地基于各种分支完成了代码开发，每个分支指向对应的一个commit之后，完成了自己本地的提交历史树的修改了，就可以将本地的提交历史树，推送到远程仓库，跟远程仓库的提交历史树进行合并

​	4、此时远程仓库会和你的本地的提交历史树保持一致

​	5、其他研发人员跟你一样，同时可以从远程仓库拉取代码，也就是拉取提交历史树，跟自己本地的提交历史树进行合并，可能会出现代码冲突。你和其他人都对同一个分支进行修改，此时你们俩的提交历史是不一样的。对你来说，一个分支是指向一个commit，对他来说，同一个分支是指向了不同的commit。此时就需要对这个分支指向的两个commit进行代码合并，可能需要解决冲突。合并之后会出现一个新的commit，就是合并了你们两个人的代码的一个commit，然后分支指向那个commit。

### 5.2.7 解决分支冲突

假设有master分支和feature2分支出现了冲突

```java
<<<<<<< HEAD
System.out.println(“I like spark......”);
=======
System.out.println(“I liike storm......”);
>>>>>>> feature2
```

​	上面的意思，就是说master分支和feature2分支出现了冲突，第一行是HEAD代码，因为HEAD目前指向master，也就是master的代码；第二行是feature2的代码。

​	这个时候，我们可以将冲突的内容删除掉，保留正常内容即可。

​	接着提交HelloWorld.java代码，就解决了冲突并且提交了代码，同时完成了feature2到master的合并。此时会自动长出来一个新的commit节点，这个commit就是解决了master和feature2冲突并且合并之后的commit。同时master指向最新的这个commit，HEAD还是指向master。

​	一般来说尽量避免冲突，让不同的人负责不同的独立的模块对应的工程，大家修改的代码都是不一样的，从根本上避免代码出现冲突

### 5.2.8 commit内幕原理深入探究流程图

 地址：https://www.processon.com/view/link/60cf1c40e401fd0e51f39fff

![git commit 深入探究](git基本操作.assets/git commit 深入探究.jpg)

# 6 基于CentOS7安装部署企业私有的GitLab服务器

## 6.1 安装GitLab

参考官方文档安装：https://about.gitlab.com/install/#centos-7

#### 1 安装并配置必要的依赖

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
#开放http访问
sudo firewall-cmd --permanent --add-service=https
#开放https访问
sudo systemctl reload firewalld
```

安装Postfix 以发送通知电子邮件

```shell
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

#### 2 添加GitLab包仓库并安装包

添加 GitLab 包存储库

```shell
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

安装 GitLab 包

```shell
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee
#https://gitlab.example.com为指定访问的url地址， 这个url需要设置为本地安装的ip地址，默认80端口无须加端口号
#例如sudo EXTERNAL_URL="https://192.168.31.101" yum install -y gitlab-ee
```

查看gitlab状态，是否安装成功

```shell
gitlab-ctl status
```

按照上面指定的url访问gitlab

重新设置密码，然后使用**root和自己设置的密码来访问gitlab**

**PS:默认管理员账户为root**

## 6.2 使用GitLab

### 6.2.1 创建group组

**PS:下列对应配置操作需要使用root管理员账户，其他账户无此权限**

![image-20210619164923558](git基本操作.assets/image-20210619164923558.png)

可选对应组的访问权限，公司内部开发建议使用private权限，只允许组内成员使用

![image-20210619165012008](git基本操作.assets/image-20210619165012008.png)

### 6.2.2 创建user用户并绑定对应组

#### 6.2.2.1 创建用户

![image-20210619165234851](git基本操作.assets/image-20210619165234851.png)

密码需要发送邮箱验证输入密码，服务器没配置邮箱验证的情况，可以通过修改用户信息来设置对应密码

![image-20210619165323987](git基本操作.assets/image-20210619165323987.png)

#### 6.2.2.2 用户绑定对应组

进入组

![image-20210619165525997](git基本操作.assets/image-20210619165525997.png)

从上到下分别为：选择用户、设置用户权限(建议设置为maintainer或者development)、确认创建用户按钮

![image-20210619165614061](git基本操作.assets/image-20210619165614061.png)

### 6.2.3 使用开发者账户关联本地git

使用开发者账户登录，并进入配置界面

![image-20210619170442740](git基本操作.assets/image-20210619170442740.png)

将本地生成的公钥添加到gitlab，点击确定

![image-20210619170518617](git基本操作.assets/image-20210619170518617.png)

将本地已有git项目关联到gitlab(也可以直接git clone GitLab上的项目(通过ssh))

```shell
git remote remove origin
#先删除本地项目关联的远程仓库(如果之前没关联，无须执行改命令)
git remote add origin gitlab项目地址
#添加gitlab远程仓库地址，gitlab项目地址就是对应项目的ssh地址，如：git@192.168.31.101:oa/oa-parent.git
git pull origin master --allow-unrelated-histories
#拉取远程仓库，并合并本地和远程两个独立启动仓库的历史
#在pull命令后紧接着使用--allow-unrelated-history选项来解决问题（该选项可以合并两个独立启动仓库的历史）
#直接clone的方式在本地建立起远程github仓库的克隆本地仓库无须执行改命令
git branch --set-upstream-to=origin/master master
#指定本地master到远程的master
#上面操作是更改了一下remote origin 所以这里要重新手动将本地master分支和远程origin/master分支做一下关联
#如果不指定的话直接pull，会出现There is no tracking information for the current branch的异常
git push -u origin master
#将本地仓库的代码推送到gitlab上去
#如果事先未执行git pull origin master --allow-unrelated-histories来合并本地和远程两个独立仓库，会报fatal: refusing to merge unrelated histories（拒绝合并不相关历史的异常）
```

引用：https://blog.csdn.net/u012145252/article/details/80628451

引用：https://blog.csdn.net/sinat_36246371/article/details/79738782

## 6.3 维护gitlab

```shell
gitlab：gitlab-ctl start
#启动
gitlab：gitlab-ctl stop
#停止
gitlab：gitlab-ctl restart
#重启
/var/log/gitlab
#gitlab日志：
gitlab-ctl tail
#查看gitlab日志：
gitlab-ctl tail nginx/gitlab_access.log
#查看gitlab对应的Nginx访问日志
gitlab-ctl tail postgresql
#查看gitlab对应的数据库postgre-sql的日志
/var/opt/gitlab/git-data
#gitlab数据存放目录
```

gitlab使用文档：http://docs.gitlab.com/ce/

# 7 工作流分析

​	**集中式工作流、功能分支工作流、GitFlow工作流、互联网公司一种GitFlow工作流的变种，每种工作流都没有绝对的好坏，主要是适用于某个场景下，某种类型的项**

##  7.1功能分支工作流

### 7.1.1 功能分支工作流概述

​	一般来说，都会准备一个master分支，作为你的稳定分支，这里的代码是随时可以上线的；有多个feature分支，每个feature分支用来开发一个功能。

​	如果线上有bug，一般分为两种，一种是影响正常用户使用流程的紧急bug，就是hotfix，比如说电商网站无法下单了，门户网站无法查看新闻了；一种是不影响正常用户使用流程的非紧急bug，比如说有某个广告显示的位置出现了偏差，就是bugfix。

​	一般线上发现了bug，就了一个hotfix或者bugfix分支，然后在分支上复现bug，修复bug，然后在测试环境进行验证以及回归测试，最后将分支合并到master去上线，修复线上问题

![image-20210617215331750](git基本操作.assets/image-20210617215331750.png)

**功能分支工作流非常适合对公司内部的系统、平台或者工具，5人以内小团队，3人左右，去开发一个比如公司内使用工具，报表平台**

## 7.2 集中式工作流

**集中式工作流：2人以内的项目，1人单独维护的一个项目**

​	某2个同学，负责维护了一个小型的工具类的系统，比如发短信/邮件/消息的这么一个工具；某2个同学，负责维护大数据计算作业类的工程，比如说spark作业，spark就是一个工程，然后在里面你要去维护一套代码，但是这套代码的话呢就是实现一套不同类型的计算任务

​	比如spark工程中，有的作业是用来分析用户行为的；有的作业是用来分析用户活跃度的

​	**不需要分支，直接每个人都是这个项目的master，都可以直接push代码，需求更新频率也很低，不是经常要修改代码的**

​	**两个人都直接基于master分支去做开发，也不涉及什么code review，pull request**

​	**基于gitlab的集中式工作流，就是这么玩儿的，说实话，因为就一两个人开发，所以不需要使用太多的gitlab的功能，就是基于一个master分支开发**

**实战一下集中式工作流：**

（1）张三和李四都在master分支上

（2）张三先修改一下代码，然后直接push到gitlab上去

（3）李四也修改代码，同时跟张三修改了同一行代码，跟张三会出现冲突，此时push失败，要求先pull，pull之后会要求解决冲突

（4）解决冲突，提交代码，再次push

![image-20210620103918081](git基本操作.assets/image-20210620103918081.png)

## 7.3 基于rebase优化集中式工作流的提交历史

​	rebase在实际工作场景中很少用，他的适用场景也有限，但是不是说rebase就没用，也有用，是有其适用的场景的

​	集中式工作流这个场景结合起来去讲解rebase，一方面是讲解你的这个rebase的原理，一方面是讲解rebase在集中式工作流场景下，使用它的作用是什么呢？

在刚才那种普通的集中式工作流开发流程下，会有一个问题，就是每次如果某个人push了代码到master，然后另外一个哥儿们也要push会失败

此时那个哥儿们就要git pull拉取和合并master分支的代码

坑爹的事情来了，每次都这样合并的话，会导致你最后发现看到这个项目，它的提交历史就是各种分叉，各种合并，提交历史感觉很难看

实际上来说，对于一个普通的集中式工作流开发的小项目，是不需要这样分叉的提交历史的，我们希望的是那种很平滑的，就一条线一样的提交历史，看起来会比较清爽

要优化集中式工作流的提交历史成为一条线，看起来非常的清晰和清爽，此时就要使用rebase命令

```shell
git pull --rebase
#rebase：变基，就是改变commit之前依赖的基础commit
```

通过git pull --rebase，执行变基式的合并，改变commit历史，看起来提交历史就是一条直线，张三先提交，李四再提交，李四再提交，张三再提交

集中式工作流的项目，看起来比较清爽

```shell
git rebase feature/001
#比如说我们有一个master分支，还有另外一个分支比如feature/001
#也可以是合并的时候，采取git rebase feature/001来合并
```

你就全部都采取git pull --rebase来跟其他人对master分支的修改进行合并，就能保持项目的提交历史就是一条直线

![image-20210620184421117](git基本操作.assets/image-20210620184421117.png) 

**PS:rebase适用于优化集中式工作流的提交历史，变成一条线，没那么多分叉，变得清爽美观**

**PS:rebase：变基，就是改变commit之前依赖的基础commit**

## 7.4 GitFlow工作流

### 7.4.1 版本稳定迭代项目的中小型团队的GitFlow工作流实战



![image-20210620190542574](git基本操作.assets/image-20210620190542574.png)

**GitFlow工作流流程**

​	1 项目刚开始建立，就两个分支，**master**分支和**develop**分支，指向的commit是一样的，代码是一样的

​	2 后续每个人从**develop**分支拉一个**feature**分支出来

​	3 对应**feature**分支开发好了，再合并到**develop**分支中

​	4 **develop**分支经过联调测试完成后，从**develop**分支拉取一个**release**分支进行QA测试

​	5 QA测试结束后，将**release**分合并到**master**进行预发布测试，**master**分支的代码在预发布环境，模拟线上的环境，进行测试和验证。

​	6 同时QA和PM，一起进行最后的验收和验证后，直接用**master**分支的代码进行v1.0版本的上线,上线之前需要对该**master**分支打一个tag，便于以后代码的回退(可以在GitLab或者命令等方式对改版本打tag)。

​	7 上线之后发现有bug，此时需要从**master**分支拉一个**bugfix**分支下来，快速进行bug复现，修复，合并到**master**和**develop**两个分支

**PS:每次master代码上线之前，都必须对master打一个标签，v1.0，v1.1。tag，标签，是什么？其实简单来说，就是指向master指向的那个commit的一个指针而已，tag指针是不能移动。说明这个commit就是一个可以稳定的可以上线的某个版本的代码**

**PS:feature分支用完了之后，是要删除掉的；release分支测试完了之后，也是要删除掉的；bugfix分支用完了之后，也是要删除掉的；唯一长期保存的只有master和develop两个分支，一个用于持续集成，一个用于稳定上线**

**GitFlow工作流适用的场景是什么**

​	适合的就是版本稳定迭代的项目，一个版本，接一个版本，接一个版本。面向企业内部的系统或者项目，OA系统，CRM系统，ERP系统，或者面向外部的系统，比如说电商系统，但是系统已经成熟稳定了，版本稳定迭代

**GtiFlow工作流的不足之处在于什么**

​	如果是做一些互联网公司里面，面向外部的重要的业务系统，业务发展的过程中，快速迭代，同时5个版本，8个版本，10个版本，**多达几十号人频繁的拉出多个版本并行开发**

​	**develop分支会处于一个极其不稳定的状态**

​	**而且不同版本的测试和开发进度不一样的**

​	比如v1.0已经集成测试完了，要进入QA测试了，但是此时develop分支还混合了v1.1版本的多个代码，你如果此时直接基于develop分支拉一个release/1.0分支，会把develop分支里面的v1.1的代码也带进去



### 7.4.2 互联网公司多版本频繁并行场景下的GitFlow变种工作流实战



![image-20210620192958558](git基本操作.assets/image-20210620192958558.png)

基于GitFlow工作流去一点点改进

整个依赖的稳定的和基准的分支只有一个，就是master分支，全部以master分支为基准和基础

（1）比如说启动一个版本

​	v1.0，涉及3个功能，投入了3个RD（resourch developer，研发工程师），直接从master分支拉3个feature分支下来，不是从develop分支拉了，也没有develop分支了；

​	同时要做一个v1.1，涉及5个功能，投入了5个RD去开发，直接从master分支拉5个feature下来，每个人就基于自己的feature去开发，搞

​	同时的话呢，每个版本，刚开始启动，除了拉一堆feature分支，还需要同时从master分支拉一个develop分支下来，专门用于这个版本的集成测试

（2）一个版本ready之后，要进入集成测试了

​	很多时候，v1.1.0这样的版本，会比v1.0.0版本先上线

​	快速发展的业务和项目，不断的招人，不断的进新人，不断的开启各种版本，经常出现v1.5.3先上线了，但是v1.2.0还没上线

​	**基准代码只有master**

​	**feature和develop都从master拉取，以master为准**

​	每个版本的RD各自迭代，各自独立开发，独立测试，加速开发和测试的进度

​	**直接跟master分支拉出来的staging分支去合并，release分支跟master分支的代码，以staging分支为基础进行合并，到这一步才进行多个版本之间的代码集成**

​	到这一步，master代码一定是稳定的，即使master分支混合了别的版本的代码，但是代码基本上都已经趋向于稳定了，所以此时进行多个版本的代码集成，风险是比较低的

​	staging代码，回归测试，QA仔细回归一遍，要把集成在一起的多个版本的代码对应的功能，全部回归测试，确保每个版本的代码对应的功能都正常运行

​	staging代码和master分支合并在一起，打tag，准备上线，这个版本就实现了上线

**PS:适用于多版本频繁并行的大型项目基于GitFlow的变种工作流，与GitFlow主要区别**

​	**1 feature改为从master拉取，只有master一个基准**

​	**2 多出一个staging分支，用于合并release和master代码，然后再讲staging合并master，相当于一个中间缓冲分支**

# 8 Git高阶实战技巧

## 8.1 比较不同分支之间差了哪些代码改动

```shell
git log --abbrev-commit feature/001..master
#或
git log --abbrev-commit HEAD..master
#可以看一下，feature/001分支比master分支落后了哪些commit，不是需要跟master保持一下同步
#HEAD就是执行当前分支最新commit的指针，当前分支是feature/001,HEAD..master和feature/001..master就是一个效果
git log --abbrev-commit HEAD..origin/master
#也可以查看当前分支和远程分支落后了哪些commit
#需注意，领先commit的分支需要放在..后面，否则命令无效果

========以下较为少用
git log --abbrev-commit feature/001 feature/002
#同时也可以看一下哪些commit是存在于两个分支中的某一个，但是不是两个分支都有的
git log --abbrev-commit feature/001 feature/002 --not master
#还可以看一下，在两个feature分支中都有的commit，但是在master分支中还没有
```



## 8.2 查看不同版本之间的代码差异具体是什么

```shell
git diff
#工作区和暂存区
git diff --cached
#暂存区和仓库
git diff HEAD
#工作区和仓库
git diff feature/001 master
#两个分支之间的差异
git diff HEAD HEAD^
#最近两次提交之间的差异
```



## 8.3 将暂存区中的多个功能代码分成多次提交

```shell
git add -i
#进入交互模式
#2 update是放入暂存区，3：revert是从暂存区里挪出来
#输入对应文件的数字，回车，对应文件就会变成*开头，就说明针对当前操作成功。再回车就推出当前操作，进入交互模式主界面
#7 quit退出交互模式
```

![image-20210620203334091](git基本操作.assets/image-20210620203334091.png)

**PS:可以通过git add -i交互模式中的revert操作将暂存区的代码挪出来，然后在分批add再commit就可以实现将暂存区中的多个功能代码分成多次提交**



## 8.4 feature分支开发到一半时切换到bugfix分支

当前分支存在没有commit的代码时，直接切换分支会警告异常，提示未提交代码会被覆盖，无法切换分支

![image-20210620204453249](git基本操作.assets/image-20210620204453249.png)

```shell
git stash
#将代码暂存起来
#暂存起来后就可以切换分支了
git stash list
#查看暂存列表
git stash apply stash@{0}
#恢复到指定的一次stash
git stash apply
#将暂存的代码恢复
git stash drop stash名称
#手动删除掉某个stash
```



## 8.5 对本地不规范的提交历史进行修改和调整

**1 修改提交日志**

```shell
git commit --amend
#直接就可以修改上一个commit的备注信息，补充更多的提交说明
#相当于是将上一个commit删除掉，然后基于上一个commit对应的代码及当前暂存区的代码重新构建一个commit object
#如果当前暂存区有代码，也会将当前暂存区的代码提交上去
```

​	场景1：如果你刚刚执行了一个commit，结果发现，不好，我其实应该补充更多的备注，才能符合公司对commit备注这块的规范的，刚才写的太简单了。git commit --amend。直接就可以修改上一个commit的备注信息，补充更多的提交说明

​	场景2：你刚执行了一个commit，结果扫了一眼代码，发现今天忘记修改几行代码了，需要修改之后补充到上一个commit中去，没关系。先修改你的代码，同时加入暂存区，然后再次执行：git commit --amend

**PS:说白了就是将上一个commit删掉，重新提交，重新提交代码自然会将暂存区的代码提交**

**PS:但是这里要注意，git commit --amend是会修改commit的SHA-1 hash值的，所以如果一个commit已经push到了远程仓库，就不要再去修改了，否则的话会造成提交历史的一个混乱**

**2 修改历史版本的commit**

```shell
git rebase -i HEAD~3
#编辑前3个版本，将对应版本的pick改成edit
#然后就回到了最早的edit版本，使用git commit --amend修改
git rebase --continue
#修改完成后，在使用git rebase --continue继续修改下一个版本
#反复使用git commit --amend修改,git rebase --continue到下一个版本，最后一次git rebase --continue就完成了所有修改
```

git rebase -i HEAD~3效果图

![image-20210620212635770](git基本操作.assets/image-20210620212635770.png)

![image-20210620212503184](git基本操作.assets/image-20210620212503184.png)

git commit --amend及git rebase --continue效果图

![image-20210620212556180](git基本操作.assets/image-20210620212556180.png)

**3 删除commit**

​	也可以通过git rebase -i HEAD~3在编辑器里删除对应commit，就实现了删除commit

**4 将多个commit合并为一个commit**

​	通过git rebase -i HEAD~3，可以将多个commit前面的命令修改为squash，就会自动将这些commit合并为一个commit

​	提示一下，如果要合并多个commit，那么一定要往前多选择一个commit

**5 将一个commit切分为多个commit**

通过git rebase -i HEAD~3，将要拆分开来的commit前面的指令，修改为edit

然后你就会回到那个commit对应的版本，此时执行git reset HEAD^，此时会直接回退到之前的状态，然后此时可以执行多次git add和git commit，形成多个commit

接着最后执行git rebase --continue结束这次commit拆分

**PS:8.5 本小节针对的全部都是本地的提交历史**

## 8.7 对本地刚做出的修改、暂存和提交进行撤回

```shell
git reset --hard HEAD
#或
git reset --hard commit的hash值
#HEAD一般是最新提交的commit指针,将本地为提交的修改变为全部撤回到该commit
#HEAD^,加一个^就是再上一个commit版本
#指定commit的hash值，就是撤回到该hash值的commit版本
git checkout -- 文件名
#就是将该文件撤回到最新commit的版本
```

**reset指令流程图：**

![image-20210621225602462](git基本操作.assets/image-20210621225602462.png)

## 8.8 远程和本地同时撤回分支合并操作

本地

```shell
git merge --abrort
#两个分支merge出现了冲突，还原到未合并的状态
git reset --hard HEAD^
#如果仅仅是在本地完成了merge后想要撤回
#很简单，直接执行,将分支挪回merge之前的那个commit即可，就相当于没执行过merge
```

远程仓库撤回上一个commit

```shell
#先
git revert HEAD
#回到了上一个commit的状态
#再
git push origin master
#同步撤回远程仓库的commit操作
```

远程撤回分支merge操作

（1）如果是要撤回已经push到远程仓库的merge操作

（2）在本地执行

```shell
git revert -m 1 HEAD
#再执行
git push origin master
```

此时本地和远程的提交历史都会多一个commit出来，该commit的内容和合并之前的master指向的那个commit是一样的，同时master此时指向最新的那个commit

（3）但是后面，如果要再次将release分支和master分支进行合并，此时是需要特殊处理的，再次在本地执行：

```shell
git revert HEAD
git push origin master
```

就是再次将master还原到指向一个新的commit，该commit的内容与上一次merge后的那个commit一样，包含merge的内容

（4）最后再次将release分支与master分支进行合并，此时可以保证release分支所有的内容，都是合并到master分支的



## 8.9 二分查找哪一次提交引入了线上bug

```shell
git bisect start
#开始二分查找
git bisect bad
#标注当前这个commit是有bug的
#输入
git bisect good
#会继续往后面二分查找；如果这个版本就有bug，那么说明是之前引入的bug
#输入
git bisect bad
#最后需要执行
git bisect reset
#就是从二分查找状态中恢复到之前的状态
```

​	**每次输入git bisect good或者git bisect bad之后，git会根据二分查找并回滚对应的git版本，用该版本进行对应测试，无bug就打good标签，有bug就继续打bad标签，直到定位到具体bug的版本**

![image-20210621220849262](git基本操作.assets/image-20210621220849262.png)

![image-20210621221310901](git基本操作.assets/image-20210621221310901.png)

**该提示信息就是当前git二分定位到的commit版本**

二分查找流程图：

![image-20210621225728064](git基本操作.assets/image-20210621225728064.png)

## 8.10 让多个项目共享一个子项目的代码修改权利

**PS:较为少用，一般一个子项目最好是独立拉一个项目版本，而不是多个项目共享**

**PS:8.10小结 欠账，未实操。。**

一般来说，submodule使用的场景，就是要在一个父项目下，再建立一个子项目。

（1）我们先建立一个DbConnector子项目

```shell
git clone git@192.168.31.80:OA/DbConnector.git
cd DbConnector
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

（2）将所有工程师的父项目中都注册DbConnector这个子项目

```shell
#进入某一个父项目，然后在其下创建一个submodule：
git submodule add git@192.168.31.80:OA/DbConnector.git
#这个时候，会出现一个新的目录，就是DbConnector。而且此时用git status命令，会发现说有.gitmodules和DbConnector两个东西等待提交。
#.gitmodules里面保存了你的子模块和其对应的git url之间的映射关系
#然后执行下面的命令，将子模块提交：
git add --all .
git commit -m ‘added DbConnector module’
git push origin master
```

接着别的研发人员，可以将这个包含了一个子模块的项目克隆到自己本地，但是此时默认是只有一个子模块的目录，里面是空的，没有文件的。

```shell
git clone git@192.168.31.80:OA/oa-parent.git
#或者是
git pull
#此时需要运行两个命令：
git submodule init
git submodule update
```

才能初始化子模块的目录，同时拉取子模块的文件

或者也可以简化一点，通过给下面的命令，一次性把上述工作都干了：

```shell
git clone --recursive git@192.168.31.80:OA/oa-parent.git
```

（3）对子项目进行代码修改

1）无论谁都可以跟平时一样，在DbConnector里面，当做是一个完全独立的项目，在里面可以修改源代码，包括说修改之后提交，分支操作，在DbConnector里面去执行就可以了

2）以后每次如果某个人要更新子模块中的数据，那么执行以下命令即可：

进入子模块的目录

```shell
git fetch
git merge origin/master
```

**2、Subtree**

**git以前是推荐使用submodule来管理子项目，但是git 1.5.2之后就推荐使用subtree了。subtree的优点，就是子项目的维护对我们来说几乎就是透明的，不需要引入太多的维护成本。**

cd P1项目的路径

```shell
git subtree add --prefix=用来放S项目的相对路径 S项目git地址 xxx分支
```

上面的代码，就会把s项目的代码（指定分支）下载到本地放入指定的目录中，同时提交到本地p1项目中

对于P2项目也做一样的事情

直接在p1中各种修改代码和提交，包括s项目的代码，都没有关系

接着某一天，p1项目负责人想把对s项目的代码更改提交到远程仓库，此时执行下面的命令：

```shell
git subtree push --prefix=S项目的路径 S项目git地址 xxx分支
```

这个时候，git会找到所有对s目录的commit，然后一起提交到其对应的远程版本库中

接着p2项目的负责人知道了这个事情，需要将s项目最新的代码更新到自己本地，此时就执行下面的命令即可：

git subtree pull --prefix=S项目的路径 S项目git地址 xxx分支

3、最后的一个总结

如果有多个仓库，多个项目，要共享对同一个子项目的修改权限的话，那么可以这么玩儿

子项目其实是有一个单独的仓库的

但是在父项目中可以为其注册子项目，接着不同仓库的人，就可以很方便的在自己的仓库代码中，修改子项目的代码，同时互相之间都可以push代码，也可以pull到别人push的代码

就算真的有类似的场景，我也不推荐使用，太复杂了

## 8.11 团队拆分迁移项目代码时简化提交历史

**PS:8.11小结 欠账，未实操。。**

将原来的一个仓库拆分成两个仓库

```shell
echo 'get history from blah blah blah' | git commit-tree 9c68fdc^{tree}
```

我们从要截断的那个commit的上一个commit，创建一个基准commit出来，比如用倒数第二个commit

```shell
git rebase--onto 622e88 9c68fdc
```

接着，执行git rebase --onto命令，将倒数第二个commit之后的commit重新基于那个commit基础之上来构建提交历史

此时，master分支代码就会只保留了2个commit

然后可以将这个master分支的代码和commit历史提交到另外一个新的代码仓库上去，那个代码仓库就会仅仅保留两个commit了

## 8.12 将新版本的功能放到上一个版本提前上线

通过合并其他分支的对应commit版本实现

```shell
git cherry-pick 其他分支的某个commit的hash
#当前分支就会合并其他分支的某个commit
```



## 8.13 基于GitLab的代码权限控制以及强制代码审查

**PS:8.13小结 欠账，未实操。。**

（1）第一件事情，有些分支是需要被保护起来的，就是说不能直接被push，尤其是master分支。

（2）仓库全都是private私有的，如果在一个部门中建立了一个项目，那么默认这个部门下的研发人员对这个项目都是有权限的，但是有的人是master，有的人是developer

（3）实践过的一个技巧：强制code review

将develop分支设置为protected

将develop分支保护起来之后，就是说，每个研发人员第一次写好feature分支的代码，有对应权限的才能合并develop分支，需要先对merge request往develop分支进行code review

review通过之后，可以合并到develop分支，做持续集成

# 9 git内幕原理

## 9.1 git目录概述

​	平时我们如果执行git init之后，就会生成一个.git目录，其中就是git存放所有数据的地方。如果我们拷贝这个.git目录，那么就相当于备份了一个完整的git项目的数据。

在.git目录中，有如下一些内容：

![image-20210626190732997](git基本操作.assets/image-20210626190732997.png)

**hooks** 目录中包含了我们的client-side或者是server-side的钩子脚本

**info** 目录中包含了我们在.gitignore文件中定义的不需要git追踪的那些被排除掉的文件

**description** 文件仅仅是由Git WebUI程序去使用的，我们一般不用去管这个文件

**config** 文件包含了这个git项目的所有配置项

其实上面那些都不是最重要的，最重要的是这几个东西：**HEAD文件，indexes文件，objects目录，以及refs目录**。

**objects**目录存储了git中所有的数据，

**refs **目录中存储了所有的指针（包括了branch，tag，remote，等等，指向了objects目录中的数据），

**HEAD** 指向了当前我们所处的分支指针

**indexes** 文件存储了暂存区中的内容。

##  9.2 提交代码的内幕原理

**PS:9.2本小节使用的git命令都是git 底层命令，并非平常使用的高级命令，高级命令也是通过使用对应底层命令的组合**

### 9.2.1 git存储机制

​	 git的数据存储格式实际上是一种非常简单的key-value存储格式。就是说，git支持将任何东西存储到其数据库中，一般就是存储文件的内容，然后用一个key值来引用它。

​	首先随便找一个目录，初始化为git项目：git init

​	此时去.git/objects目录中检查一下，只有两个子目录，info和pack，没有任何文件，此时数据库是空的。

![image-20210622211816274](git基本操作.assets/image-20210622211816274.png)

**使用底层命令对git进行操作**

```shell
echo 'hello world' | git hash-object -w --stdin
#默认的表现是接受一个value，然后针对这个value返回一个hash值，默认是不将value写入自己的数据库的
#-w之后，git就会将value写入数据库之中，同时用hash值作为key来引用那个value对应的文件。同时--stdin指的是从命令行接收value，也就是echo ‘test content’中的test content
git hash-object -w 123.txt
#如果不使用--stdin，那么要求是在hash-object之后跟一个文件名，它就会将一个文件作为value存储到数据库中去，然后返回一个hash值
#git返回的是SHA-1 hash值，是40位的字符串，是根据文件内容计算出来的一个checksum，校验和。
#123.txt做了修改，再使用hash-object -w存储一遍就会生成新的hash版本
```

​	git会将一个文件的多个不同版本，以完整快照的方式存储在objects数据库中，每个版本都用一个单独的文件来存储，每个版本的文件都有一个不同的hash值

### 9.2.2 git轻量级索引机制

**1 git将40位hash值的头2位作为目录，其实可以认为是一种轻量级的索引机制**

![image-20210624065405971](git基本操作.assets/image-20210624065405971.png)

​	40位hash值就可以来引用这个文件。存储的时候，40位hash的头2位，会作为目录名称放在objects里面，然后那个存储的文件会放在那个目录中，文件名是40位hash值的后38位。

​	这样的话，有一个好处，就是每次你要根据一个hash值定位一个数据的时候，直接根据头2位先定位到一个目录，然后再在这个目录下去查找，linus写git的时候，就是用了一种非常轻量级的文件索引机制

**2 从git中恢复被删除的文件**

```shell
rm -rf test.txt
git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
#恢复被删除文件
#恢复被删除的文件需要指定对应文件名,是否能够不知道文件名直接恢复，推测git 恢复版本，推测需要其他组合命令
```



### 9.2.3 查看存储在git中的文件内容

直接查看git objects中的源文件是乱码，需要通过对应命令查看

```shell
git cat-file -p ee2b8364542e6512c6ff1d906cfa3787731f3ec2
#查看对应文件内容
git cat-file -t ee2b8364542e6512c6ff1d906cfa3787731f3ec2
#查看git中存储的文件的类型
#git中存储的文件的类型，默认都是blob
```

![image-20210624072313738](git基本操作.assets/image-20210624072313738.png)

​	**通过查看库中文件类型可以和之前对应的原理关系串起来了，实际的文件的内容，每个文件的版本快照，会作为一个blob object存储在git数据库中**

扩展(linux命令)

```shell
echo 'version 1' > test.txt
#使用改命令可将对应字符串覆盖test.txt中的文件内容
find .git/objects -type f
#可查找对应目录的所有文件并展示
```

### 9.2.4 tree object

git有另外一种object叫做tree object，每次将一堆文件的一个版本存储到git仓库中，都是用blob object来存储那些文件的版本，然后用一个tree object来引用多个blob object。tree object通常会包含指向其他多个blob的指针，或者是其他tree object的指针。

```shell
git cat-file -p master^{tree}
#查看master分支指针指向的那个tree object
```

git通常是基于暂存区中的内容来创建一个tree object的，我们首先需要在暂存区中加入一些内容，然后可以试着手动创建一个tree object。

```shell
git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt
#git update-index：底层命令，一般是git add命令去调用的，将数据库中存储的某个blob数据的引用放到暂存区中
#--add：因为暂存区中还没有这个文件，你可以认为这个文件是第一次进入暂存区，所以需要这个选项
#--cacheinfo：不是从工作区中加入文件放到暂存区，是从git仓库（git数据库）中加入文件
#100644 mode：这种模式的意思是，这个是一个普通的文件
```

接着使用下面的命令基于暂存区中的内容，创建一个tree object：

```shell
git write-tree
#将暂存区中的内容创建为一个tree object，tree object也是放入了git仓库，主要是包含了对一个或多个blob obejct的引用
```

![image-20210624194529294](git基本操作.assets/image-20210624194529294.png)

下面的命令可以从仓库中读取一个tree object放入暂存区，然后再基于这个tree object创建一个新的tree object出来：

所谓的将仓库里的东西读出来放到暂存区，实际上只是将一些引用放到暂存区，但是blob，tree，实际还是存储在objects中的

```shell
git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
git write-tree
git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
```

此时tree object包含两个文件和一个tree object，如果将这个tree object的数据恢复到工作区中，就会有两个文件和一个叫做bak的文件夹，bak文件夹中也包含了一个文件。

### 9.2.5 commit objcet



```shell
echo 'first commit' | git commit-tree d8329f
#基于已有的tree object创建一个commit object，同时给这个commit object一个提交备注
```

![image-20210624195220410](git基本操作.assets/image-20210624195220410.png)

​	commit object的含义很简单，就是指向了一个tree object，而那个tree object指向了多个blob，其实这个tree object就代表了某一时刻项目中所有代码文件的快照版本，同时那个commit object包含了作者、提交时间、提交备注，此外还包含了指向上一次commit object的指针。

### 9.2.6 提交代码内幕原理总结

​	**git add：**相当于是将你的有修改的文件作为一个新的版本，采用单独的blob文件存储到仓库中去**(git hash-object -w)**，同时在暂存区中加入这些最新版本的blob的引用，替换同一个文件之前在暂存区中的版本**(git update-index --add --cacheinfo 100644)**

​	**git commit：**相当于就是将暂存区中的文件快照，也就是对应的blob指针创建一个tree object**（git write-tree）**，作为项目此时此刻的一个快照版本，放入仓库，同时再创建一个commit object包含了提交人/提交时间/提交备注**（git commit-tree d8329f）**，来指向那个tree object，代表了一次提交对应的仓库快照

**PS:blob、tree和commit，三种object都是作为一个独立的文件在objects目录中存储的，存储时的子目录命名和文件名，都是基于hash值来的**

**PS:SHA-1 hash值如何计算:首先blob的内容分为两块，一个是header，一个是content。header就是类型+空格+长度+\u0000。然后会用header+content来计算出一个SHA-1校验和，作为hash值**

​	1 git update-index --add 文件名：其实就是，我们在工作区里修改了文件以后，会用这个底层命令，将工作区中有修改的文件的一个版本，放入git仓库中作为一个blob去存储这个版本，同时将这个blob的引用放入暂存区中

​	2 多次有git update-index之后，git仓库中会存储项目中每个文件的所有版本，同时在暂存区中保留了每个文件的最新版本，没有变化的就保留值钱的版本

​	3 执行git write-tree，就可以针对暂存区中当前项目的最新版本，创建一个tree object，引用项目中所有的文件当前此时此刻的一个版本对应的blob，tree object，就代表了项目当前的一个完整快照版本

​	4 接着执行git commit-tree，就可以基于这个tree object创建一个commit object，包含了tree object的创建人，创建时间，备注信息，等等，作为一次提交

**PS:每个commit，就是当前项目的一个完整的快照版本，包含了项目的所有文件的一个最新版本**

## 9.3 git指针内幕原理

### 9.3.1 git/ref

​	.git/refs目录中，有几个子目录，包括了heads和tags两个子目录，在这里就有各种指向objects中的指针。

​	.git/refs/heads目录下，会包含很多个文件，就是每次创建一个分支，就会在这个里面对应新建一个文件，文件名称就是分支的名称，文件里的内容就是分支指针当前指向的那个commit object的hash值，代表着这个分支当前指向了哪个commit object

```shell
echo 6257d0304b12daddd8984ef1e82648385bdad88b > .git/refs/heads/master
#创建master的分支指针
git log --pretty=oneline master
#查看master的commit object的提交历史
```

![image-20210624201835020](git基本操作.assets/image-20210624201835020.png)

下图对应master创建后的详细信息

![image-20210624201951029](git基本操作.assets/image-20210624201951029.png)



```shell
git update-ref refs/heads/master  6257d0304b12daddd8984ef1e82648385bdad88b
#更新某个refs文件指针指向的commit object
#在.git平级目录指向，且目录不能带上.git,会有异常
```

![image-20210624203007906](git基本操作.assets/image-20210624203007906.png)

```shell
git update-ref refs/heads/test a9fe1ecdc
#创建一个test分支，让其指向某个commit object
```

​	实际上我们如果执行git branch命令，就是执行update-ref命令，创建那个分支对应的文件，然后文件内容就是对应的commit object的SHA-1 hash值。

​	删除一个分支很简单的，就是删除.git/refs/heads下面的分支对应的目录和文件即可

### 9.3.2 HEAD文件

HEAD文件中，保存了对某个分支指针的引用，其实就是某个分支对应的文件。

HEAD其实也是一个文件，但是也是一个指针，这个HEAD呢，就是指向了当前你所处的那个分支而已，保存的是refs/heads下面的分支对应的目录和文件名

![image-20210624203425286](git基本操作.assets/image-20210624203425286.png)

从比如master分支切换到test分支，实际上就是一个非常轻量级的操作

（1）HEAD文件里的内容，从refs/heads/master，变为refs/heads/test，代表着HEAD指针指向了test分支

（2）将test分支指向的那个commit对应的tree，对应的那些blob对应的文件版本的内容，一次性从仓库里恢复到工作区中，工作区中所有文件的版本和内容，会变成那个分支指向的commit当时的那个快照版本

每次切换分支之后，HEAD都会指向那个分支的文件

```shell
 git symbolic-ref HEAD refs/heads/master
 #切换HEAD指针
```

![image-20210624203824639](git基本操作.assets/image-20210624203824639.png)

**PS:分支就是一个文件，保存了指向的commit的40位hash置。HEAD也是一个文件，保存了指向的那个分支对应的文件名称**

### 9.3.3 tag object

​	其实除了blob、tree和commit三种object之外，还有一种object就是tag object。tag object是指向某个commit object的，包含了标签时间，标签名称，等等。而且tag object永远就指向创建时指定的那个commit object，不能修改。

我们可以通过下面的命令创建一个轻量级的tag object

​	就只是一个指针而已，在refs/tags下面，会有一个tag名称对应的文件，然后里面就是那个tag指向的commit的40位hash值

```shell
git tag v1.0
#轻量级的tag，直接就指向了当前分支指向的那个commit object，就是在refs/tags下面搞一个文件，作为一个指针而已
#底层命令相当于
git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d
#底层命令
git tag -a v1.1 -m 'test tag'
#重量级的tag,创建一个带备注的标签
#此时查看cat .git/refs/tags/v1.2，会发现返回的不是我们指定的那个commit object，是另外一个新创建的tag object的SHA-1 hash
```

**PS:tag不是一个独立的分支，tag只是附属于某个分支的tag标签。拉取远程分支的话需要将tag所属的分支拉取关联。对应远程分支的提交历史就有tag了。然后可以再创建一个本地分支来关联这个tag。**

![image-20210624205338158](git基本操作.assets/image-20210624205338158.png)

### 9.3.4 remote

​	如果我们添加了一个remote，同时push了一些东西到那个remote，git会在refs/remotes中保留对每个分支，最近一次push到远程服务器对应的commit。

比如下列命令

```shell
git remote add origin git@192.168.31.101:YoungStar/123.git
git push -u origin --all
git push -u origin --tags

```

​	查看一下上面的文件，会看到一个SHA-1 hash值，就是这个分支最近一次push到服务器的object commit的SHA-1 hash值。

![image-20210624210116060](git基本操作.assets/image-20210624210116060.png)

**FETCH_HEAD**，就是最近一次fetch下来的每个分支对应的commit hash值，就在这个里面

![image-20210624210319766](git基本操作.assets/image-20210624210319766.png)

### 9.3.5 总结

**分支：.git/refs/heads**

**HEAD：.git/HEAD**

**tag：.git/refs/tags**

**remote：.git/refs/remotes**

**FETCH_EHAD：.git/FETCH_HEAD**

## 9.4 git文件合并内幕原理

### 9.4.1 文件合并打包packfile实操

​	哪怕是很大的文件，比如说20kb的一个代码文件，每次改动一点代码，git也是存储改动后的版本对应的一个完整的文件；如果这个20kb的代码文件修改了100次，会怎么样？存储100个版本对应的文件，也就是100个文件，2000kb ~ 2MB。所以这样的话对存储的压力还是蛮大的。。。。

​	git实际上是可以更加智能的处理这种文件改动的。默认情况下，git会使用loose格式存储文件（loose，松散格式，采取这个文件的每个版本，就存储一个完整的单独的文件），但是一段时间过后，git会将多个loose格式的松散文件打包到一个packfile二进制文件中，来节省磁盘空间。

​	什么时候git会进行packfile打包的操作呢？两个时机：我们手动执行git gc操作，或者是将文件push到远程服务器的时候，git会看一下loose格式文件是不是太多了，如果是，则进行打包合并

```shell
git gc
#手动packfile打包命令
```

git gc打包前后对比图

![image-20210624212037046](git基本操作.assets/image-20210624212037046.png)

​	会发现出现了objects/info/packs和objects/pack等新的目录。

**PS:git gc：garbage collector，jvm gc是不一样的，git用来处理自己的文件的**

**PS:如果我们将本地的commit数据push到了远程服务器后，就代表远程服务器包含了我们的数据了，此时git就可能会执行一次packfile打包**

​	我们是将之前所有的hash值组成的文件打包成了一个packfile文件，进行了压缩以及合并

​	pack包含了两个文件，一个是packfile包含了打包的所有数据，一个是index文件，包含了每个blob object在packfile中的offset，通过.idx文件标识出每个object在.pack文件里是哪一个部分

​	git做文件存储的优化，两块：多个文件打包成给一个，压缩，二进制格式，packfile，节省空间；在打packfile的时候，就会对我们刚才说的那种情况，大ruby文件，仅仅保留第一个版本对应的完整内容，然后之后的版本都是直接保留的是它的那些delta增量内容，后面的版本就不是全部保留全量内容了，进一步节约空间

```shell
git commit -am 'modified repo a bit'
#直接将工作区中的修改过的文件加入暂存区，再执行一次提交， 不常用
```

​	**git，git gc；git push，自动会做打packfile**

​	实际上，在git进行打包packfile的时候，就会对一个文件的多个版本，仅仅保留其每次增量修改数据，避免对一个文件保留多个全量的版本快照。

```shell
git verify-pack -v .git/objects/pack/pack-1a31d6539ef87a1390ee8271f48591cca98310d3.idx
#查看包的.idx文件标识出每个object在.pack文件里是哪一个部分
#或
 git verify-pack -v .git/objects/pack/pack-1a31d6539ef87a1390ee8271f48591cca98310d3.pack
#两个命令查看效果相同
```

![image-20210624212359806](git基本操作.assets/image-20210624212359806.png)

​	会发现ruby大文件，第一个版本仅仅包含9kb的内容，但是第二个版本包含了22kb的内容，所以这里就做了增量处理和优化，仅仅对一个大文件的最新的版本保留了全量的内容，但是对之前的版本都是增量内容

### 9.4.2 packfile打包总结

（1）git gc / git push，会执行packfile的压缩优化

（2）将.git/objects下的各种object对应的文件，压缩成一个packfile

（3）每个packfile都对应一个.pack文件和一个.idx文件，.pack文件包含了多个object的内容，.idx文件包含了每个object在那个.pack里面的offset偏移量

（4）在打packfile的意义：多个文件合并一个，节省空间：对大文件进行增量处理，仅仅是最新的版本保留全量内容，之前的版本保留增量内容

（5）用git verify-pack -v命令，可以查看.pack文件里包含的具体内容

## 9.5 git远程分支内幕原理

​	对于远程仓库的分支，本地都是有一个对应的分支是追踪那个远程分支的(orgin/master)，同时本地还有一个分支是跟本地追踪分支关联起来的(master)

​	本地的master，是跟本地的origin/master关联起来的，作用是在git pull和git push

**git pull，相当于是两个命令：git fetch origin + git merge origin/master**

​	相当于就是将远程仓库的提交历史拉取下来，然后合并在一起，本地的origin/master指向最新的commit，接着将本地的master与origin/master进行合并

1）将远程仓库的提交历史拉取到本地进行合并，一个commit可能会出来多个分叉

2）同时将origin/master跟远程仓库的master指向一个commit

3）将本地的master和origin/master进行合并，可能需要解决冲突

**执行git push:**

​	会将本地的origin/master指向master指向的那个指针，同时会将本地提交历史推送到远程进行合并，然后远程仓库的master分支会指向最新的那个commit

每次执行git remote add命令之后，实际上就会在.git/config文件中加入一段配置，类似下面

```
[remote "origin"]
	url = git@192.168.31.101:YoungStar/123.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

​	git checkout -b feature/001 origin/feature/001，这段命令执行之后，就会在.git/config中加入[branch]配置，标注了本地分支跟origin/*追踪分支之间的关联关系

```
[branch "master"]
	remote = origin
	merge = refs/heads/master
[branch "test"]
	remote = origin
	merge = refs/heads/test
```

**refspec**

​	就是fetch后面的那串东西， +refs/heads/*:refs/remotes/origin/*，就是**refspec**

​	refs/heads/*，代表的是远程仓库被追踪的分支，远程仓库的master

​	refs/remotes/origin/*，代表的是本地仓库用来追踪远程仓库的分支，本地仓库的origin/master分支

​	+，代表的是在执行git pull的时候，不是要将远程分支和本地分支进行merge么？即使不是fast-forward的类型，是3-way merge也是要去执行的

​	在执行git remote add命令的时候，本地的git客户端会执行fetch命令获取远程仓库的refs/heads/*下面的分支，然后将其写入本地的refs/remotes/origin/*下面去。所以之前说的那些本地追踪远程的分支，比如origin/master之类的，实际上指的就是refs/remotes/origin/master代表的分支

#### fast forward

假设当前只有一个master主分支，最新节点为c1
创建并切换到新的分支dev后
一直在dev分支工作 最新节点为c9
若mater在dev被创建后一直停留在c1节点
则此时将dev直接合并到master
即直接将c9节点作为master的最新节点 就称为 **fast forward**

引用：https://www.cnblogs.com/baebae996/p/12971067.html

Three-way merge：

![image-20210624220015160](git基本操作.assets/image-20210624220015160.png)



引用：https://www.zhihu.com/question/30200228/answer/866309494

是可以手动指定多个**refspec**的（不常用）

```
[remote "origin"]
    url = https://github.com/schacon/simplegit-progit
    fetch = +refs/heads/master:refs/remotes/origin/master
    fetch = +refs/heads/qa/*:refs/remotes/origin/qa/*

[remote "origin"]
    url = https://github.com/schacon/simplegit-progit
    fetch = +refs/heads/*:refs/remotes/origin/*
    push = refs/heads/master:refs/heads/qa/master
```

# 10 维护开源项目及贡献代码

**PS:10 小结 欠账，未实操。。**

1 发布开源一个项目
（1）start a project

（2）将自己本地的一个项目代码上传到github上去

git remote add origin https://github.com/roncoo-eshop/test-project.git

git push -u origin master

（3）管理collabrator，这些人是你的开源项目的重要合作伙伴，他们是可以审核别人提交的pull request代码合并请求的

（4）你自己当然平时可以写代码了

你当然是可以自己拉个什么分支，在那个什么写代码，但是为了维护开源项目的版本

（5）比如你现在要搞一个v1.0版本，拉一个对应的分支出来

（6）接着，包括你在内的所有人都依据下面的流程来对v1.0这个版本进行贡献

2、给开源项目贡献代码

​	如果要给某个开源项目进行贡献，通常都是在GitHub上，fork一个开源项目，fork之后这个项目就会有一份副本存在于我们自己的命名空间中，这样我们不需要开源项目的push权限也可以贡献代码了

（1）fork出来一份，在自己的账号里有一个拷贝的项目，同时将自己的ssh key加入github，将fork出来的项目克隆到本地

（2）从fork出来的项目的master分支创建一个自己的分支出来

（3）在那个分支上提交代码

（4）push分支和代码到自己fork出来的项目

（5）在github上提交一个pull request，对开源项目提交

# 11 学习指南

（1）如果说你想继续去看，同时磨炼自己读英文文档的能力，《Pro Git》英文版，看一遍，慢慢看，提升你的英文文档的学习能力

（2）git版本不断在变化的，官网 看每个版本的更新的新功能，release notes，看每一个新功能增加了什么东西

（3）你可能会在公司里去做项目，碰到一些问题：找我请教：跟其他同学交流：百度+google，报错，搜索

（4）在公司里大量实践git，自己去搭建gitlab服务器，整个项目选择一种合适的开发工作流去做，日常碰到各种git问题和场景，用高阶技巧去处理，自己反复理解git底层的内幕原理





























