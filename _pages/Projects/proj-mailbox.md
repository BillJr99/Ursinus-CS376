---
layout: assignment
permalink: Projects/Mailbox
title: "CS376: Operating Systems - Interprocess Communication via a Mailbox"
excerpt: "CS376: Operating Systems - Interprocess Communication via a Mailbox"

info:
  coursenum: CS376
  points: 100
  goals:
    - To copy data between user and kernel memory space
    - To manipulate process state to block and unblock processes until events occur
    - To implement an interprocess communication mechanism via a mailbox

  rubric:
  - weight: 80
    description: Code Functionality
    preemerging: Test cases or reproduction steps are not given or are not appropriate
    beginning: A reasonable attempt is made with feasible test cases documented in the README
    progressing: Code works and appropriate test cases given in the README pass with minor adjustments 
    proficient: Code works when the patch or source files are applied, and testing succeeds with the appropriate steps given in the README, along with my own test cases
  - weight: 20
    description: Code Documentation and README
    preemerging: Both the README and documentation are lacking
    beginning: Either the README or documentation are lacking
    progressing: A README and code documentation are provided 
    proficient: A README and code documentation are provided and appropriate  

  readings:
    - rlink: "https://wiki.tldp.org/static/kernel_user_space_howto.html#Implementation-8"
      rtitle: "Kernel Space - User Space Interfaces"

tags:
  - linux
  - kernel
  - project

---

You will write a kernel module or component that enables us, via syscalls, signals, or other mechanism, to communicate between two processes.  You will implement a mailbox with `send` and `receive` capabilities.  `receive` shall be a blocking call, which blocks until a message is received.

Implement the following functions:
* `mysend(pid_t pid, int n, char* buf)`
* `int myreceive(pid_t pid, int n, char* buf)`

In these functions, `pid` is the pid you wish to send to/receive from.  

## Receive Syscall
When receiving, if `pid` is set to a negative number, `myreceive` shall receive from any process.  `n` specifices the number of bytes to send, and the maximum number of bytes to read.  `buf` is the buffer to send, and is a pointer to a char buffer that `myreceive` will assume has been properly `malloc`'ed.  

But, these pointers are addresses that exist in user-space, not in kernel space!  You'll need to use `copy_from_user`, `copy_to_user` and `strlen_user`, to help you (after all, these addresses are virtual user-space addresses, not the physical addresses we need).  Read [this article](https://wiki.tldp.org/static/kernel_user_space_howto.html#Implementation-8) for details on how to use these functions, or see the example below taken from that article.  `myreceive` returns the number of bytes read, or -1 on error. 

```c
// From: https://wiki.tldp.org/static/kernel_user_space_howto.html#Implementation-8
// GNU Free Documentation License

#include <linux/linkage.h>
#include <asm/uaccess.h>
asmlinkage long sys_mysyscall(char __user *buf, int len)
{
    char msg [200];
    if(strlen_user(buf) > 200)
        return -EINVAL;
    copy_from_user(msg, buf, strlen_user(buf));
    printk("mysyscall: %s\n", msg);
    copy_to_user(buf, "hello world", strlen("hello world")+1);
    return strlen("hello world")+1;
}      
```

## Send Syscall

For this project, you are only required to start out by receiving messages that have already been sent.  That is, a call to `mysend` before you call `myreceive` should result in the message being missed.  However, you can implement your calls in one or more of two ways to resolve this:

1. Have `mysend` block until `myreceive` receives the message.
2. For extra credit (and in addition to the option above), allow `mysend` not to block but instead queue the message so that a late call to `myreceive` will not block, but rather deliver one of the queued messages.  In this case, if `myreceive` is called and no message is available, return `0` as the number of bytes read rather than blocking.  You can specify a parameter to `myreceive` that indicates if it should block or use the queue, or write a separate version of `myreceive`.

## Testing Your Syscalls

To test, implement a thread barrier using `fork`'ed children that communicate their arrival to one another using messages.  Once any one process has received `N-1` messages (from the others), it sends a release message to all the other processes to release them.  The user processes can print a log message right before and right after they block (perhaps with their PID).