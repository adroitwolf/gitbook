---
title: git的使用心得
date: 2021-02-08 15:48:26
tags:
- git
categories:
- git
---

# git的使用心得

由于考研失利，最近重新拾起了java项目。在学spring cloud的时候想到了单点登录，之后就想到了spring security OAuth 2。于是在大改我的blog项目之前，先要弄一个example源码试试看。但是之前总是用git的可视化界面，现在想装个B,用命令行编写的时候，突然发现自己的命令全忘了，于是学习了一会git命令。并且用这个博客记录下来。


---------

## 本地电脑连接远程github
其实这个做法主要就是在你本地生成ssh密钥，然后将公钥给你的远程账号就ok了。



1.  下载和配置Git

去官网下载个git，之后再命令行界面type git出现一堆结果就证明成功了。

```bash
git config --global user.name "YOUR NAME"

git config --global user.email "YOUR EMAIL"

```

网上都说下载完之后要配置名字邮箱，不过这两个和你的git账号没有关系，随便写就行。


> 更新与3/1/2021,其实上面的那个邮箱还是有点用的，如果你这里配置的用户名和邮箱和你的git账号不一致的话，git push的时候你的Git账号里面是没有contribution的

2. 生成 ssh 
首先我们要有我们的ssh命令，在windows里面生成ssh的rsa密钥，具体命令如下：

```bash
ssh-keygon -t rsa -c ""
# 引号里面是邮箱，外国人有毛病，什么玩意都要弄个邮箱，我就直接为null了

```

生成的ssh文件一般都在 ~/.ssh/里面

[![yUg8pj.png](https://s3.ax1x.com/2021/02/08/yUg8pj.png)](https://imgchr.com/i/yUg8pj)


我们打开我们的git-> settings -> ssh.... 
将我们的pub公钥上传上去就ok了

> 注意，git项目中有三种方式可以下载下来：
> 1. HTTPS(这个和上面的ssh没有关系，第一次push还是要输入面)
> 2. SSH
> 3. zip包

## git创建过程中得基本命令

我首先在自己得本地创建了idea工程，之后去git里面创建仓库repos。
其实，git里面有新手引导，我们一步步得看。
[![yUR3yn.png](https://s3.ax1x.com/2021/02/08/yUR3yn.png)](https://imgchr.com/i/yUR3yn)


首先进入我们得仓库目录，创建README文件，之后git init 进行初始化仓库。


> git branch -M main  这个意思是将你本地得分支改名字，我这里初始得就是master，我就不改了

我这里由于idea有add功能么，我就没有add我的idea文件，直接add的readme,之后就是commit 进行提交。

> git remote add origin xxxxx 这个意思是添加一下这个仓库可以remote的路径.注意 remote里面可以存多个url，毕竟你的git和gitee可以同时存仓库。 origin是在存在本地的名字，相当于id，到时候我们直接pull或者Push的时候直接带着id就可以了。



后面我们发现git push 后面加了一个-u 这个是绑定一个默认远程仓库，以后还想要push这个仓库直接git push命令即可。

## .gitignore

有的时候，我们并不想把一些ide之类的自动生成的文件push上去，所以我们可以编写一个忽略文件。

但是有的时候写上这个之后，发现并不起作用，所以我们需要清空git的缓存，并重新添加。

```bash
git rm -r --cached .
```


## 多个git项目配置


需求： 当我们在不同的项目中可能需要不同的git账户，或者需要不同的用户名和邮箱。

解决方法：
**方法一**
打开项目的.git/config文件夹，增加下面的内容即可：
```text
[user]
	name = xxx
	email = xxx@xxx.com
```
**方法二**
```bash
git config user.name "YOUR NAME"

git config  user.email "YOUR EMAIL"

```
> 注意 这个和之前全局的名字邮箱命令相比少了--global，也就是全局配置文件。这个是局部配置文件里面修改的。