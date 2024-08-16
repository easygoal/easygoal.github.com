---
layout: post
category: guideline
tagline: ""
keywords: server
tags: ["guideline","git"]
header:
lang: zh_CN 
date: 2019-05-24
title: Use repo to manage multiple git repositories
---

# 引导脚本
repo安装前需要首先下载引导脚本,

```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
国内用户被GFW屏蔽的话, 可以使用清华大学tuna提供的git-repo镜像替代,
```bash
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo
```
打开下载的引导脚本, 在开始可以找到如下信息,

```bash
REPO_URL='https://code.google.com/p/git-repo/'
REPO_REV='stable'
```
这个url是repo本身仓库的地址, 在后续执行"repo init" 的时候会先clone这个仓库. 我们这里可以先将它clone下来, 以便后续将这个仓库推送到自己本地建立的repo仓库里(如果前面下载的是tuna的repo镜像, 这里的url就不是google的地址).
```bash
git clone --bare https://code.google.com/p/git-repo/
```

# 搭建仓库
有了引导脚本, 就可以搭建仓库了. 这里假设我们在本地已经有了一个git账户, 而且已经将当前账户的ssh public key拷贝到了该git账户下. 我们后续的所有实验, 都是针对本地的git仓库实施的. 实际搭建仓库的时候需要根据远程服务器的具体地址进行修改, 这里不再赘述.

使用repo需要事先建立两个仓库, 一个用来管理repo代码本身, 另一个管理manifest文件.
```bash
mkdir git-repo.git
cd git-repo.git
git init --bare
 
mkdir manifests.git
cd manifests.git
git init --bare
```
现在我们可以将之前clone好的repo代码仓库, 迁移到我们自己建立的git-repo仓库里.
```bash
cd git-repo.git
git push --mirror git@localhost:/home/git/repo-test/git-repo.git
```
接下来需要编写manifest文件, repo要根据这个文件的信息来管理所有git仓库. 首先需要将前面建好的manifest仓库clone下来, 并在里面建立default.xml文件.
```bash
git clone git@localhost:/home/git/repo-test/manifests.git
cd manifests.git
touch default.xml
```
 写入相应的内容, 并提交到远程,

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remote name="origin"
          fetch="." />
 
  <default remote="origin"
           revision="master" />
 
  <project path="repo-demo/foo" name="foo" />
  <project path="repo-demo/bar" name="bar" />
</manifest>
```
 这里假设我们事先有了两个名为foo.git和bar.git的仓库. project项指的就是我们的仓库, path指的是执行"repo sync"后远程仓库被clone下来后的本地路径, name指的是仓库的原始名.

另外, remote项指的是manifests仓库的地址. name指代远程仓库名, fetch指的是url. 这里fetch可以写绝对路径, 也可以写相对路径. 如果你在服务器上的代码仓库, manifests仓库与repo仓库都在同一级目录下(这也是常见做法), 那么fetch地址可以写".", 表示在manifests仓库所在的目录下寻找其他project仓库. repo会自己拼凑出正确的project仓库url.

到目前为止, 我们自己搭建的repo仓库就已经可以使用了, 尝试sync一下,

```bash
repo init -u git@localhost:/home/git/repo-test/manifests.git
repo sync
repo start master --all
```

# repo without gerrit
