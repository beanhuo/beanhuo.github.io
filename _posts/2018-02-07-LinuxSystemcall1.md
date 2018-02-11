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
System call getpid() is defined to return an integer that is the current process’s PID. The implementation of
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
