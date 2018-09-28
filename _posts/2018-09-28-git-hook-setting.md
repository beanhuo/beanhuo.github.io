---
layout: post
title: "如何配置Git发送git-commit 邮件通知"
date: 2018-09-28   
tag: Git 
---

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


