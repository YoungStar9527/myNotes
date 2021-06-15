# 1 git绑定github

## 1.1 下载git，安装git，一直下一步即可，傻瓜式操作

## 1.2绑定github

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

## 1.3 本地git目录绑定，上传github

### 1.3.1 建立远程仓库

![image-20210530103403458](git基本操作.assets/image-20210530103403458.png)

输入远程仓库名,及打勾README file文件，初始仓库需要一个默认文件，才能绑定

![image-20210530103459406](git基本操作.assets/image-20210530103459406.png)

### 1.3.2 本地仓库绑定远程仓库

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

### 1.3.3 修改github默认展示分支

可以将master设为默认展示分支

![image-20210530104036222](git基本操作.assets/image-20210530104036222.png)

## 1.4 gitignore配置

引用：https://www.cnblogs.com/xuanjiange/p/13458618.html
