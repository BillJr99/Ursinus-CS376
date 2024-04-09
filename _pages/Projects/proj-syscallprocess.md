---
layout: assignment
permalink: Projects/SyscallProcess
title: "CS376: Operating Systems - System Calls and Processes in the Linux Kernel"

info:
  coursenum: CS376
  points: 100
  goals:
    - To manipulate kernel data structures that manage tasks
    - To explore the permissions model used by the Linux operating system
    - To create system calls to expose functionality from the kernel to the user
    - To test kernel functionality with wrapper programs

  contract:
    a:
      - "Complete the system call for tasks 5-6 (6 is an A+)"
      - "Complete the wrapper program for task 5-6 (6 is an A+)"
      - Complete the writeup as part of the README
    b:
      - Complete the wrapper programs for tasks 1-2
      - Complete the system calls for tasks 3-4
      - Complete the wrapper programs for tasks 3-4
    c:
      - Complete the system calls programs in tasks 1-2
    d:
      - Complete the tutorial system call

  readings:
    - rlink: https://tldp.org/HOWTO/html_single/Implement-Sys-Call-Linux-2.6-i386/
      rtitle: "Implementing a System Call or Linux 2.6 for i386"
    - rlink: https://linuxgazette.net/133/saha.html
      rtitle: "Learning about Linux Processes by Amit Saha"
    - rlink: https://litux.nl/mirror/kerneldevelopment/0672327201/app01lev1sec4.html
      rtitle: "Traversing Linked Lists (in Linux)"
    - rlink: https://www.linuxjournal.com/article/5833
      rtitle: "Kernel Locking Techniques by Robert Love"

tags:
  - linux
  - kernel
  - project
  - syscall
  - processes

---

The following projects in this course will involve your directly manipulating and modifying kernel data structures.  Naturally, only the kernel can do this, but for simplicity of testing, we'd like you to use user-level programs to test your code.  Normally, you wouldn't want to expose so much to the user (there are only a couple hundred total syscalls!), but this will make things easier in the scope of our course, without loss of anything academic.  This is akin to calling read() or write() from a user program -- you're calling code in the kernel, but you can only do so using the interface that it provides you.  That interface is a collection of functions known as system calls (or syscalls).  You will write a series of syscalls for these projects, and test them using user programs (which I will call wrappers).  First, a tutorial on creating a system call is helpful (code examples are thanks to Dr. Haungs of Cal Poly State University and [this article](https://tldp.org/HOWTO/html_single/Implement-Sys-Call-Linux-2.6-i386/)).

## Tutorial: Creating a System Call

To create a system call, you start by simply writing a function in the kernel somewhere.  This can be an existing file (like `kernel/sched.c`), or a new file that you create.  If you create a new file, you'll need to add it to the Makefile so that it gets built when you compile.  Either way is fine.  Here's a simple implementation of a syscall to get the PID of the current process.  The syscall here would be called `mygetpid` to the user, but in the kernel it is referred to as `sys_mygetpid`.

```c
asmlinkage long sys_mygetpid(void)
{
  return current->tgid;
}
```

Copy the function header prototype to `include/linux/syscalls.h` so that you can call this by name.  You can paste it below the other function prototypes, at the bottom of the file (but right before the `#endif`).  As a reminder, the function prototype looks like this: `asmlinkage long sys_mygetpid(void);`

Saving this in a file in the `kernel/` source directory (`sched.c` would be fine), we must now open this function up to the user.  To do this, we add it to the list of available system calls.  This list is found in a file called `arch/i386/kernel/syscall_table.S` At the end of this file, add the line

`.long sys_mygetpid`

Syscalls are referred to by number rather than by name internally, so we must give this syscall a number.  To do this, edit the file `include/asm-x86_64/unistd.h` and add the following to the blank space near line 633 of the file (which is the end of the rest of this list).  You can get to this line in `vim` by typing `633gg` (which means to "go to line 633"), and then open insert mode by pressing the letter `o` to enter the following lines:

```c
#define __NR_mygetpid ^^^ //(where ^^^ is one more than the highest number in the list).
__SYSCALL(__NR_mygetpid, sys_mygetpid)
```

The syscall number for your first syscall should be `285`, since the current "last item" in the list is `sys_eventfd` with number `284`.  You'll increment this number each time you make a new syscall.

Build and install your kernel (don't forget to `make install` and `update-grub` if you are working directly within the virtual machine!), and reboot, to make your syscall active in the kernel.  As a reminder, this is how you build your kernel (from within the `linux-2.6.22.19` directory):

```
make -j2 EXTRAVERSION='.19-LASTNAME' C=0
su
make install
update-grub
reboot
```

We're ready to test the function.  To do this, create a user program, **outside of the kernel source directory** called `testSysCall.c`, in which we call the syscall as follows:

```c
#include<stdio.h>
#include<unistd.h> /* Has mygetpid syscall number */

int main()
{
   int syscallnum = 285; // use your syscall number from earlier here
   printf("Process ID: %d\n", syscall(syscallnum)); // alternative to mygetpid() if this function is not exposed by the kernel to user-space
   return 0;
}
```

You can write this in your home directory **outside of the linux-2.6.22.19 directory** since this is now a user program that invokes your system call by its system call number.  If you are modifying and compiling the kernel within your virtual machine, you can call `mygetpid()` as your system call; if you compiled your kernel and booted it from outside the virtual machine, then the header files you modified aren't availbale here, so you can use the `syscall(syscallnum)` approach.  Later, if your system call has parameters, you can simply add them like this: `syscall(syscallnum, parameter1, parameter2)` as appropriate. 

Compile the user program with `gcc -I/The/location/of/your/kernel/include testSysCall.c`, and run as normal on your virtual machine booted with your custom kernel. If you simply invoke your syscall by number, i.e., `syscall(285)`, you can omit the `-I` flag.  For example, if your linux source code is in your `user` home directory and called `linux-2.6.22.19`, you would compile with `gcc -I/home/user/linux-2.6.22.19/include testSysCall.c`.  This gives your program access to the Linux kernel header files which you edited with your new system call number and function prototype.

## What to Do

Now that you know how to create a syscall in the kernel, you will create a few syscalls and, for each, **write a user wrapper program to test the syscall**.

### Task 1: Elevate a Process to Root Privileges

Create a system call to set a process ID's `task_struct`'s `uid` and `euid` to `0`, thus claiming the process by the root user (giving the process root priveleges).  This is an awful thing to do in reality, but we will try it here to give you an idea of the power that comes with manipulating the kernel (and the care that you must take in doing so!).  

Call your syscall `steal`, and it should take in a single parameter of type `pid_t` (the process ID number to elevate to root).  Write a wrapper user program that takes in a process ID as a command line paramter, and invokes the syscall to elevate it to root.  Your wrapper program will have to refer to pid as a `long` (not an `int`!). To convert `argv[1]` to a long, use the `long atol(char*)` function.

Try running it on your bash shell and then run a new instance of bash; you will see your prompt go from `$` to `#`, indicating you are now root!  No `su` password required.  To get the pid of your bash shell, run the `ps` command.

Your syscall should return `0` on success, and a token number on failure.  You may need to start a new bash task to verify this (i.e., type `bash` again at the prompt to run a new instance of the shell as root).

#### Searching for a Task by PID

In the tutorial, we operated on the current task, so instead of searching for the `task_struct` by its PID, we could simply use the global variable `current`.  Here, you will want to search for a `task_struct` variable from the list of running tasks, given its PID.  [This article](https://linuxgazette.net/133/saha.html) describes how to search the linked list of running tasks.  There is a macro defined to loop over this list, which you can invoke as follows:

```c
struct task_struct *task;
for_each_process(task)
{
    // if task->pid is equal to the pid you're searching for
    // then task is the task_struct you are seeking!
}
```

#### User Test Program

For reference, here is the test user program for this system call.  Run this with `./a.out <your bash pid number>`.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/syscall.h>

#define EXIT_FAILURE -1
#define EXIT_SUCCESS 0

int main(int argc, char *argv[]) {
    // Check if the command line argument count is correct
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
        exit(EXIT_FAILURE); // A token value for a failure condition
    }
    
    // Convert argv[1] to a long to represent the PID
    long pid = atol(argv[1]);
    
    // Check for conversion error
    if (pid <= 0) {
        fprintf(stderr, "Error: Invalid PID.\n");
        exit(EXIT_FAILURE);
    }
    
    // Attempt to invoke the syscall to elevate the process to root
    // Note: The syscall number is assumed to be 286 for demonstration purposes.
    // This syscall number and functionality are hypothetical and not part of standard Linux distributions.
    long result = syscall(286, pid);
    
    // Check the result of the syscall
    if (result == 0) {
        printf("Process %ld elevated to root successfully.\n", pid);
    } else {
        perror("Error elevating process to root");
        exit(EXIT_FAILURE);
    }
    
    return EXIT_SUCCESS;
}
```

### Task 2: Quadruple the Timeslice of a Given Process

Create a system call called `quad` that takes a process ID, retrieves its `task_struct`, and quadruples its current timeslice.  Return the new timeslice on success, and a token number on failure.

To find the timeslice field, see the `task_struct` definition in `include/linux/sched.h`, whose definition begins on [line 821](https://elixir.bootlin.com/linux/v2.6.22.19/source/include/linux/sched.h#L821).  The name of the field can be found on line 851.

#### Testing

Write a wrapper user test program as you have before.  You will find it helpful to have a long-running program whose pid you can use to pass as input to this wrapper.  I suggest writing a program called `spin` that consists of a `main` function with a `while(1);` infinite loop. You can run this in the background as many times as you like, and run `ps` to get the pid of those spinning processes.  This approach will be helpful on this and the upcoming tasks.  You won't notice much at first glance when you test system calls like these, but if you run `top` you should see how much CPU time each `spin` program is consuming, and you'll likely see them adjust after the system call!

### Task 3: Swipe the Timeslice from Another Process

Create a system call called `swipe` that takes a process ID called `target` and another process ID called `victim`, and as long as `target != victim`, takes the timeslice from the victim (setting `victim`'s timeslice to 0), and adds it to the `target`.  Also take all `children != target` from the victim and take their timeslice, too.  Return the amount of timeslice taken on success, and a token number on failure. 

Note that the `task_struct` contains a list called `children` that you will use to iterate over its children (which are stored in a list of `sibling`s of one another).  You can do this using the following pattern adapted from [Linux Kernel Development by Robert Love](https://litux.nl/mirror/kerneldevelopment/0672327201/app01lev1sec4.html).

```c
struct list_head *p;
struct task_struct *child;

// assumes you already have a task_struct* victim defined
list_for_each(p, &(victim->children)) {
        child = list_entry(p, struct task_struct, sibling); // the next child is a sibling of this child, not the child of this child!
        // your code here
}
```

### Task 4: Turn a Process into a Zombie

Write a syscall called zombify that also takes a process ID called `target`, and sets the task's `exit_state` (see line 869 of `include/linux/sched.h`) to `EXIT_ZOMBIE`.  You can test this one by running it, and then separately running `top` to observe your target program.  

Take a detailed look at the [`do_exit()` function in the kernel](https://elixir.bootlin.com/linux/v2.6.22.19/source/kernel/exit.c#L862).  Note that this function does other things besides simply setting the task's state to `EXIT_ZOMBIE` (which it does on line 842 in `exit_notify()`, which is called by `do_exit()`.  Why are these other steps necessary in practice, and what happens differently when you use your syscall instead?  

For example, one thing `do_exit()` does is send a `SIGCHLD` to its parent.  This is done via the call to `exit_notify()` from within `do_exit()`.  Why is this important (again, contrast this behavior with what happens in your version of `zombify`). In spite of this, why do you think the process is reaped by the parent anyway on a call to `wait()`?

### Task 5: Implementing Thread Join

Write a syscall called `myjoin` that also takes a process ID called `target`, and becomes `TASK_UNINTERRUPTIBLE` until that process exits. You will want to do this by using the `sleep_on` and `wake_up` family of methods.  

You will know when the process exists because it will call `do_exit()` (which already exists in `kernel/exit.c`) -- so `do_exit()` can check to see if a pid is joined (create a data strucutre of your choice to maintain this information).  You may restrict your implementation such that at most one process may join to a given process.  You can add this information to the `task_struct` data structure in `sched.h`, and initialize any values in `do_fork()` in `kernel/fork.c`.  **Note that the process is called `p` in `do_fork()`, and `tsk` in `do_exit()`, as opposed to `current`, since these functions manipulate those structures!**

When a process exits, `do_exit()` should make the source pid state `TASK_RUNNING` again.  Note that you should do some error checking here -- the target pid must exist, and must not be dead, a zombie, or otherwise terminated before executing `myjoin`.  You will need to [lock](https://www.linuxjournal.com/article/5833) using `lock_kernel()` and `unlock_kernel()`, to ensure that the task doesn't finish during your call to `myjoin`, as well! 

Here is a pseudocode outline of the approach:

```
add a struct task_struct* joiner field to the task_struct, and add a line to do_fork() to set this field to null
add a wait_queue_head_t joinqueue to task_struct, and add a line to do_fork() to init_waitqueue_head(&(current->joinqueue))

in your join system call:
    lock_kernel()

    check if the target pid is null (unlock the kernel and return if it is)

    get the target pid task_struct

    check if the target->exit_state field is EXIT_ZOMBIE (unlock the kernel and return if so)

    check if target->joiner is not null (unlock the kernel and return if so)

    set target->joiner = current
    
    unlock_kernel()

    interruptible_sleep_on(&(target->joinqueue)) // or set current->state to TASK_UNINTERRUPTIBLE
    
in do_exit:
    if current->joiner is not null, set it to null
    wake_up_interruptible_sync(&(current->joinqueue)); // or set current->joiner state to TASK_RUNNING
```

Test this program by writing and running a program that sleeps for 20 seconds and then returns.  Write a second program to join to it, print a log message, and then return.  That log message should not appear until after the join returns; that is, after the sleeping program terminates after approximately 20 seconds.

### Task 6: Forcewrite

Write a syscall called `forcewrite` that, given an `int` file descriptor (much like `sys_write`), writes to a file.  The twist is that, in `forcewrite`, you will not check for file permissions (or you will ignore them) before writing to the file.  

You can test this by creating a file (manually) on your file system, and making it read-only.  Use `forcewrite` to write to the file anyway.  You should refer to `sys_write` as a reference implementation, and copy liberally from this function to implement your own syscall.  You might omit some calls from this function, or make copies of and modify subroutines that it calls in your implementation.  

Write a paragraph detailing how your version of `forcewrite` differs from the kernel stock version of `sys_write`, and, more importantly, why you think it differs from the stock version (i.e., what changes did you make exactly and why).
