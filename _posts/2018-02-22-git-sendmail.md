---
layout: post
title: "用git提交patch，并发送邮件"
date: 2017-02-22   
tag: Git
---

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

## get maintainers

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



