---
layout: post
title: "git send-email tricks"
date: 2015-02-22   
tag: Git
---


# 用git提交patch，并发送邮件   

### 安装 git git-email
> apt-get install git git-core git-email

### .gitconfig

```
[user]
name = xxx
email = xxx@xxx.com
[sendemail]
smtpencryption = tls
smtpserver = smtp.gmail.com
smtpuser = xxx@gmail.com
smtpserverport = 587
```

###  cleanfile
如果需要，在提交之前， run ./script/cleanfile xx.c

### git commit
提交我们的修改。

### git format-patch
> git format-patch -n --cover-letter 

-n 的意思 是生成n个patch，从 HEAD 往前。
如果要邮件标题前加入标记可以用--subject-prefix

>git format-patch -numbered -s --cover-letter --subject-prefix="PATCH v2" 

### checkpatch.pl
> ./scripts/checkpatch.pl 0001-xxxx.patch

### get maintainers

> ./scripts/get_maintainer.pl ./*.patch


### 发送邮件
> git send-email 00000x-sssss.patch --to xxx@gmail.com --to 2@gmail.com --cc ddd@gmail.com


如果想要编辑patch邮件内容，加--annotate选项
编辑完一个退出vim用:wn命令,编辑下一个patch，直到最后一个直接wq退出vim.

如果我们经常要给一个固定的邮件组提交patch,我们可以用git-email的 --identity功能，首先
在.gitconfig中建一个新的sendemail组，如下：

```
[sendemail "lkml"]
	smtpserver = smtp.gmail.com
	smtpuser = myuser@gmail.com
	smtpencryption = tls
	smtpserverport = 587
	chainreplyto = false
	cc = LKML <linux-kernel@vger.kernel.org>; ddd <ddd@gmail.com>
	to = maintainer1@xxx.com,maintain2@xxx.com

```

之后我们就可以用下面的命令方式来提交PATCH
> git send-email --identity=lkml  ./*.patch


# 如何配置Git发送git-commit 邮件通知    

### 什么是git hook
Git hook 是用来探测当前git 仓有没有人提交更新到本git Server,如果有新的git push,git hook会根据
本git hook的配置发送邮件到每一个接收者。
默认情况下， git hook是禁止的， 存在在每一个git　仓下的.git/hook下， 当然也可能通过core.hooksPath
修改。eg: git config core.hooksPath 新的hook地址。
可以查看：https://git-scm.com/docs/githooks

### 具体步骤

#### 1.设置邮件
在 git server 仓的.git/config下设置如下：

```
[hooks]
    showrev = "git show -C %s; echo"
    emailprefix = "[your-email-project-prefix] "
    mailinglist = "abc@example.com,xxx@ddd.com"
    envelopesender = send@sender.com
```
#### 2. 修改你的git项目名
当发送邮件到你时， 你希望知道具体是哪一个项目的更新提交。
可以通过修改.git/description，通过邮件接收者。

```
vi .git/description 
project-name  #内容只包含项目名。
```
如果不作修改， 在你的邮件内会有如下的提示：

```
This is an automated email from the git hooks/post-receive script. It was
generated because a ref change was pushed to the repository containing the
project "UNNAMED PROJECT".
```
通过修改后， 邮件的主题出示如下的格式：
> [your-email-project-prefix] project-name branch master updated. [the SHA1]

#### 3. 使能 git hook
对于这一步， 只要更改在.git/hooks/下的post-receive.sample 为post-receive。
同时加上可执行权限。 chmod +x.

