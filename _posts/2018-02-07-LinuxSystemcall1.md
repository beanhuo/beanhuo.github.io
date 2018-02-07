---
layout: post
title: "Linux System call example"
date: 2017-02-07   
tag: Syscalls 
---

### What is Syscalls?
System calls provide a layer between the hardware and user-space processes, which serves three primary purposes:
> 1. Providing an abstracted hardware interface for userspace.      
> 2. Ensuring system security and stability.
> 3. A single common layer between user-space and the rest of the system allows for the virtualized system provided to processes.
In Linux, system calls are the only means user-space has of interfacing with the kernel and the only legal entry point into the kernel other than exceptions and traps. Other interfaces, such as device files or /proc, are ultimately accessed via system calls. Interestingly, Linux implements far fewer system calls than most systems.

### Syscalls getpid()
Let's have a look at how Linux implement getpid() system call, then we add one our user defined system call.

system call getpid() is defined to return an integer that is the current process’s PID. The implementation of
this syscall in the kernel(kernel/sys.c) is simple:

>SYSCALL_DEFINE0(getpid)
>{
>        return task_tgid_vnr(current);
>}

SYSCALL_DEFINE is a macro, which defind in include/linux/syscalls.h.

>#define SYSCALL_DEFINE0(sname)  
>	SYSCALL_METADATA(_##sname, 0); 
>	asmlinkage long sys_##sname(void)
also, sys_getpid(void) need to be predefined in the inlcude/linux/syscalls.h

>asmlinkage long sys_getpid(void);




Ref:
[1]: https://notes.shichao.io/lkd/ch5/


Mac OS X上安装Git**      

提供两种方法参考：      

> 1、通过homebrew安装Git，具体方法请参考[homebrew的文档](http://brew.sh/)      
> 2、直接从AppStore安装Xcode，Xcode集成了Git，不过默认没有安装，你需要运行Xcode。     

**在Windows上安装Git**      

> 从[https://git-for-windows.github.io](https://git-for-windows.github.io) 下载，然后按默认选项安装即可，安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！


### 配置Git      

安装完成后，还需要最后一步设置，在命令行输入：

>* $ git config --global user.name "Your Name"
>* $ git config --global user.email "email@example.com"

"Your Name"： 是每次提交时所显示的用户名，因为Git是分布式版本控制系统，当我们push到远端时，就需要区分每个提交记录具体是谁提交的，这个"Your Name"就是最好的区分。          

"email@example.com"： 是你远端仓库的email       

--global：用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然我们也可以对某个仓库指定不同的用户名和Email地址。         



### 开始使用-建立仓库：

你在目标文件夹下使命令：    

>* git init  （创建.git文件）      

就会创建一个 `.git` 隐藏文件，相当于已经建立了一个本地仓库。

**添加到暂存区：**      

>* git add .   （全部添加到暂存区）    
>* git commit -m ' first commit'  （提交暂存区的记录到本地仓库）     


### 其它   

git branc 查看时如出现

>*  (HEAD detached at analytics_v2)   
>*  dev
>*  master

代表现在已经进入一个临时的HEAD，可以使用 `git checkout -b temp` 创建一个 temp branch，这样临时HEAD上修改的东西就不会被丢掉了。
然后切换到 dev 分支上，在使用 git branch merge temp，就可以把 temp 分支上的代码合并到 dev 上了。



