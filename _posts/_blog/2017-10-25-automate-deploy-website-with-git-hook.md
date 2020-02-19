---
layout: post
title:  "使用Git Hook实现自动部署"
categories: blog
tags: ['git']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

腾讯云到期，切到了阿里云，重新做 git hook 时竟然忘记了，记录一下。

## 背景

搭建了静态博客，少不了向服务器推送静态文件，每次都使用 `scp` 命令实在有些难受，而且还发现使用scp向服务器推送文件夹时往往覆盖不了，很是难受(`{source: 'macOS', target:'ubuntu'}`)。

于是就看看有没有方便的方法，比如ftp客户端，但是感觉还是需要手动去替换文件，有没有自动化的东西呢？查一查，果然是有的，有一个叫 `git hook` 的神奇魔法。

## 用法

1.首先在服务器初始化一个远程 git 裸库

```
git init --bare blogRepo
```

2.配置 git hook

```
cd /blogRepo/hooks/
cp post-update.sample post-update

vi post-update
```
打开 `post-update` 文件，修改内容为以下部分

```
#!/bin/sh
unset GIT_DIR 
DIR_ONE=/root/blog／  #此目录为服务器页面展示目录 
cd $DIR_ONE
git init
git remote add origin ~/blogRepo
git clean -df
git pull origin master
```

3.本地库添加/修改远程仓库地址

```
# 添加
git init
git add remote origin username@remote_ip:remote_dir
git push origin master

#修改
git remote set-url origin username@remote_ip:remote_dir
git push origin master

```

## 结果

现在在需要向服务器推送博客的时候，只需要执行如下命令就可以了。

```
# 生成静态博客页面
cd /blog_jekyll
bundle exec jekyll b

# 推送到服务器
cd /_site
git add .
git commit -m 'add post'
git push origin master
```

## 思考

现在其实是两步走，编写 `博客.md` 后，先执行生成静态网站命令，在执行推送命令，那么可以优化为一个命令，那么如何集成为一步？

## 参考资料

[用 Git 钩子进行简单自动部署-凹凸实验室](https://aotu.io/notes/2017/04/10/githooks/index.html)

[使用 Git Hook 实现网站的自动部署](https://dearb.me/archive/2015-03-30/automate-deploy-your-websites-with-git-hook/)

{% endpost #9D9D9D %}





