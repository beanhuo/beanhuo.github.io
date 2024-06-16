---
layout: post
title: "Linux System call example"
date: 2017-02-07   
tag: Syscalls 
---

### What are Syscalls?

System calls (syscalls) are the mechanism through which user-space applications interact with the kernel. They provide an abstracted interface for processes to request services and resources from the operating system. Syscalls serve three primary purposes:

1. **Providing an Abstracted Hardware Interface**: Syscalls abstract the complexities of the underlying hardware, offering a simplified interface for user-space applications. This abstraction allows developers to interact with the hardware without needing detailed knowledge of its specifics.

2. **Ensuring System Security and Stability**: By controlling access to hardware resources and critical system functions, syscalls help maintain system security and stability. The kernel can enforce permissions and access control, ensuring that only authorized processes can perform certain actions.

3. **Providing a Common Layer for Virtualization**: Syscalls offer a unified interface for user-space processes, making it easier to implement virtualization. This common layer allows the kernel to manage resources efficiently and present a consistent environment to applications.

In Linux, syscalls are the primary means by which user-space processes interface with the kernel. They are the only legal entry point into the kernel, aside from hardware exceptions and traps. Even other interfaces, such as device files or the `/proc` filesystem, are ultimately accessed through syscalls. Interestingly, Linux implements fewer system calls than many other operating systems, relying on their flexibility and extensibility.

### Example: The `getpid()` Syscall

Let's examine how the `getpid()` syscall is implemented in Linux. The `getpid()` syscall returns the process ID (PID) of the calling process. The implementation is straightforward and can be found in `kernel/sys.c`.

#### Implementation of `getpid()`

The `getpid()` syscall is defined using the `SYSCALL_DEFINE0` macro:

```c
SYSCALL_DEFINE0(getpid) {
    return task_tgid_vnr(current);
}
```

#### Understanding the `SYSCALL_DEFINE0` Macro

The `SYSCALL_DEFINE0` macro simplifies the declaration and definition of syscalls. It is defined in `include/linux/syscalls.h`:

```c
#define SYSCALL_DEFINE0(sname) \
SYSCALL_METADATA(sname, 0); \
asmlinkage long sys##sname(void)
```

This macro does the following:

1. **Defines Syscall Metadata**: The `SYSCALL_METADATA` macro is used to associate metadata with the syscall, including its name and the number of arguments (in this case, 0).

2. **Declares the Syscall Function**: The macro declares the syscall function using the `asmlinkage` keyword, which ensures the correct calling convention is used. The function name is constructed by concatenating `sys` with the syscall name (e.g., `sys_getpid`).

#### The `task_tgid_vnr(current)` Function

The `getpid()` implementation calls the `task_tgid_vnr(current)` function, which returns the thread group ID (TGID) of the current process. In Linux, the TGID is the same as the process ID for single-threaded processes, so this function effectively returns the PID.

#### Predefining the Syscall Function

The syscall function (`sys_getpid()`) must be predefined in `include/linux/syscalls.h`:

```c
asmlinkage long sys_getpid(void);
```

### Adding a User-Defined Syscall

To add a custom syscall, follow these steps:

1. **Define the Syscall Function**: Implement the function in a kernel source file (e.g., `kernel/sys.c`).

2. **Use the `SYSCALL_DEFINE` Macro**: Define the syscall using an appropriate macro (e.g., `SYSCALL_DEFINE0`, `SYSCALL_DEFINE1`, etc.) based on the number of arguments.

3. **Declare the Syscall Function**: Predefine the function in `include/linux/syscalls.h`.

4. **Register the Syscall**: Update the syscall table to include the new syscall. This usually involves editing an architecture-specific file (e.g., `arch/x86/entry/syscalls/syscall_64.tbl` for x86_64 architecture).

### Example: Adding a Custom Syscall

Let's add a custom syscall named `my_syscall` that takes no arguments and returns a fixed integer value (e.g., 42).

1. **Define the Syscall Function** in `kernel/sys.c`:

```c
SYSCALL_DEFINE0(my_syscall) {
    return 42;
}
```

2. **Declare the Syscall Function** in `include/linux/syscalls.h`:

```c
asmlinkage long sys_my_syscall(void);
```

3. **Register the Syscall** by updating the syscall table. For x86_64, edit `arch/x86/entry/syscalls/syscall_64.tbl` and add:

```
__NR_my_syscall     548     sys_my_syscall
```

(The syscall number `548` is just an example. You need to choose an appropriate number that is not already used.)

4. **Recompile the Kernel**: Build and install the updated kernel.

5. **Test the Custom Syscall**: Write a user-space program to invoke the new syscall:

```c
#include <unistd.h>
#include <sys/syscall.h>
#include <stdio.h>

#define __NR_my_syscall 548

int main() {
    long result = syscall(__NR_my_syscall);
    printf("my_syscall returned %ld\n", result);
    return 0;
}
```

This program will call the custom syscall and print the returned value (42).

By following these steps, you can implement and test custom syscalls in the Linux kernel, extending its functionality to meet specific needs.




Ref:
[1]: https://notes.shichao.io/lkd/ch5/
