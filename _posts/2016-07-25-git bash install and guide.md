---
layout: post
title: git 使用配置（windows版本）
---

下载git，地址：[https://git-scm.com/downloads](https://git-scm.com/downloads "git下载地址") 
下载完成后安装。

安装工具tortoisegit，地址[https://tortoisegit.org/](https://tortoisegit.org/ "下载地址")

安装完成两个工具后，在任意位置右键git bash here，如图：
![示例图片](/article_images/git_bash_here.png)

输入以下命令：

    git config --global user.name [username]
	git config --global user.email [email]
	使用git config --list查看已设配置

并且可以配置git免密码登录，命令如下：

	git config --global credential.helper store

此时将会在用户~目录下创建一个.gitconfig文件，内容如下：

	[user]
        name = xxx
        email = xxx@gmail.com
	[credential]
        helper = store

此后输入一次密码后，下次可以直接push、pull代码到本地，使用很方便。