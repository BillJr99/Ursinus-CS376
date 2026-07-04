---
layout: assignment
permalink: Projects/Mailbox
title: "CS376: Operating Systems - Interprocess Communication via a Mailbox"

info:
  coursenum: CS376
  points: 100
  goals:
    - To copy data between user and kernel memory space
    - To manipulate process state to block and unblock processes until events occur
    - To implement an interprocess communication mechanism via a mailbox
    - To implement kernel linked lists and dynamic memory allocation

  rubric:
    - weight: 10
      description: Modifications to the Kernel Data Structure
      preemerging: No modifications made or modifications completely break the system functionality.
      beginning: Incomplete or incorrect modifications to the kernel data structure, with major issues.
      progressing: Adequate kernel data structure modifications with minor issues or missing documentation.
      proficient: Comprehensive and correct modifications with clear documentation and rationale for changes.
    - weight: 10
      description: Copying Data Between Kernel and User Space
      preemerging: No data copying between kernel and user space is present or it completely fails.
      beginning: Copying of data is error-prone, with potential security or stability risks.
      progressing: Data copying is functional but may have minor inefficiencies or lack some safety checks.
      proficient: Efficient and error-free copying of data, with safety checks and validation.
    - weight: 10
      description: Data-level Thread Safety and Atomic Instruction Code Blocks with Locking Techniques
      preemerging: No consideration for thread safety, leading to severe system instability.
      beginning: Insufficient thread safety measures, with a high risk of race conditions or deadlocks.
      progressing: Most operations are thread-safe, with some potential for race conditions or deadlocks.
      proficient: Thread safety ensured with atomic operations and appropriate use of locking mechanisms.
    - weight: 15
      description: System Call Implementation
      preemerging: System calls are not implemented or are completely non-functional.
      beginning: System calls are implemented with significant flaws or misunderstanding of requirements.
      progressing: System calls are mostly correct but may have minor issues with edge cases or error handling.
      proficient: System calls are implemented correctly with proper parameter handling and error checking.
    - weight: 10
      description: System Call Integration with the Operating System
      preemerging: System calls are not integrated, rendering the system calls unusable within the OS.
      beginning: Poor integration of system calls, causing instability or incorrect behavior in the OS.
      progressing: System call integration works, with some minor inconsistencies or deviations from standards.
      proficient: Seamless integration, with system calls functioning as expected within the OS environment.
    - weight: 10
      description: User Space Test Program
      preemerging: No user space test program is provided, or it fails to execute any meaningful tests.
      beginning: Test program is inadequate, with unclear output and failure to cover necessary scenarios.
      progressing: Test program covers most scenarios but may miss some edge cases or lack detailed output.
      proficient: Comprehensive test program covering all cases, with documentation and clear output.
    - weight: 10
      description: Robust and Defensive Code Implementation
      preemerging: Code is not robust, with no attempt to handle errors or edge cases.
      beginning: Code lacks robustness, with frequent errors or crashes under edge cases.
      progressing: Code is generally robust but may not handle all edge cases or lacks some defensive measures.
      proficient: Code is robust, handles all edge cases, and includes defensive programming practices.
    - weight: 15
      description: Condition Variable Implementation
      preemerging: No implementation of condition variables, or implementation causes system failure.
      beginning: Condition variables are implemented incorrectly, leading to inefficient or incorrect operations.
      progressing: Condition variables are used appropriately, but there may be minor inefficiencies or delays.
      proficient: Correct use of condition variables, with efficient handling of wait and signal operations.
    - weight: 10
      description: Code Documentation and README
      preemerging: Both the README and documentation are lacking.
      beginning: Either the README or documentation are lacking.
      progressing: A README and code documentation are provided.
      proficient: A README and code documentation are provided and appropriate.

  readings:
    - rlink: "https://web.archive.org/web/20220705071626/https://wiki.tldp.org/static/kernel_user_space_howto.html#Implementation-8"
      rtitle: "Kernel Space - User Space Interfaces"
    - rlink: "https://www.oreilly.com/library/view/linux-device-drivers/0596000081/ch10s05.html"
      rtitle: "Kernel Linked Lists"
    - rlink: https://litux.nl/mirror/kerneldevelopment/0672327201/app01lev1sec4.html
      rtitle: "Traversing Linked Lists (in Linux)"
    - rlink: https://www.linuxjournal.com/article/5833
      rtitle: "Kernel Locking Techniques by Robert Love"
    - rlink: https://tldp.org/HOWTO/html_single/Implement-Sys-Call-Linux-2.6-i386/
      rtitle: "Implementing a System Call or Linux 2.6 for i386"
    - rlink: https://litux.nl/mirror/kerneldevelopment/0672327201/ch11lev1sec4.html
      rtitle: "kmalloc()"
    - rlink: https://archive.kernel.org/oldlinux/htmldocs/kernel-api/API-list-for-each-entry-safe.html
      rtitle: "list_for_each_entry_safe()"  

tags:
  - linux
  - kernel
  - project

---

> **Core Concepts — why this matters.** This project reinforces the essential OS topics of *interprocess communication (IPC)*, *the user/kernel memory boundary*, *blocking and waking processes*, and *kernel data structures and locking*. You will build a mailbox using the kernel's intrusive linked list and wait queues — read the [Kernel Linked Lists, Locking, and File Data Structures](../KernelDataStructures) reference first, as it walks through `struct list_head`, `container_of`, and the spinlock-vs-mutex decision using this very mailbox as its worked example.

You will write a kernel module or component that enables us, via syscalls, signals, or other mechanism, to communicate between two processes.  You will implement a mailbox with `send` and `receive` capabilities.  `receive` shall be a blocking call, which blocks until a message is received.  There is **no barrier** in this project: the only synchronization requirement is that `myreceive` blocks until a message arrives (and, optionally, that `mysend` blocks until its message is received). You do not need to make all senders and receivers rendezvous at a common point — each send/receive pair stands on its own.

Implement the following functions:
* `void mysend(pid_t pid, size_t n, __user char* buf)`
* `long myreceive(size_t n, __user char* buf)`

Here, `pid` is the pid you wish to send a message to.  Look up the `task_struct` corresponding to that pid.  I recommend writing a function to find a `task_struct` by its pid to help you:

```c
struct task_struct *find_task_by_pid(pid_t pid) {
    struct task_struct *task;
    for_each_process(task) {
        if (task->pid == pid) {
            return task;
        }
    }
    return NULL;
}
```

You will create (perhaps in `include/linux/sched.h`) a `struct` containing a `char*` message and an `int` size of the message.  Let's call it `mymessage`.  Add a linked list to `task_struct` of these `mymessage` data structures.  You can initialize it in `do_fork`.  Similarly, add a `wait_queue_head_t` to `task_struct` in order to wait for a message to arrive, and initialize this in `do_fork` as well.

## Receive Syscall
When receiving, `n` specifices the number of bytes to send, and the maximum number of bytes to read.  `buf` is the buffer to send, and is a pointer to a char buffer that `myreceive` will assume has been properly `malloc`'ed.  

## Send Syscall

For this project, you are only required to start out by receiving messages that have already been sent.  That is, a call to `mysend` before you call `myreceive` should result in the message being missed.  However, you can implement your calls in one or more of two ways to resolve this:

1. Have `mysend` block until `myreceive` receives the message.
2. For extra credit (and in addition to the option above), allow `mysend` not to block but instead queue the message so that a late call to `myreceive` will not block, but rather deliver one of the queued messages.  In this case, if `myreceive` is called and no message is available, return `0` as the number of bytes read rather than blocking.  You can specify a parameter to `myreceive` that indicates if it should block or use the queue, or write a separate version of `myreceive`.

## Testing Your Syscalls

To test, write a program that `fork`s a child and sends a `char*` message to that child.  The child can receive and print the message.

## What to Do

You can make modifications directly to `sched.c` and `sched.h` to add the logic for `mysend` and `myreceive` system calls, including any necessary data structures for message queuing.  I suggest the following workflow:

### Add a `mymessage` data structure in `sched.h` 

Define a structure to represent a message and a queue to hold messages for a process.  This struct should contain the `pid_t` of the sender, the `size_t` size of the message, and a `char*` to hold the address of the message itself.  

### Create a Kernel Linked List of these `mymessage` structures

Add a `struct list_head` to the `mymessage` structure, and create a `message_queue` that contains a `struct list_head` and a `wait_queue_head_t` for blocking on `myreceive`.

This data structure might look like this:

```c
struct mymessage {
    pid_t sender_pid;
    size_t size;
    char* msg;
    struct list_head list;
};

struct message_queue {
    struct list_head messages;
    wait_queue_head_t wait;
};
```

### Modify the `struct task_struct` in `sched.h` to include a pointer to your `struct message_queue` structure called `msg_queue`

You can add this pointer anywhere within the `struct`.  Initialize these in `do_fork` like this, after the call to get a `task_struct* p` returned from `copy_process`, and inside the error check statement `if(!IS_ERR(p))` that ensures that this process was established correctly:

```c
// These statements assume you've named your variables as we have above; adjust as needed
p->msg_queue = kmalloc(sizeof(struct message_queue), GFP_KERNEL);
INIT_LIST_HEAD(&(p->msg_queue->messages));
init_waitqueue_head(&(p->msg_queue->wait));
```

### Create System Calls for `mysend` and `myreceive` in `sched.c` 

These will contain the logic to append a message to a destination task's message queue (denoted by the `pid`), and to receive a message from your own current message queue from a particular specified `pid` (or from any process if the `pid` parameter is `-1`).  `myreceive` returns the number of bytes read, or `-1` on error.  

Here are the function prototypes for `mysend` and `myreceive`:

```c
asmlinkage void sys_mysend(pid_t pid, int n, const char __user *buf);
asmlinkage long sys_myreceive(int n, char __user *buf);
```

### Register your System Calls

Add entries for your two system calls to the syscall table, as before.  As usual, copy your function prototypes to `include/linux/syscalls.h`, and add an entry to `arch/i386/kernel/syscall_table.S` with an entry `.long sys_mysyscall` with the name of each of your syscalls.  To call these syscalls by name, add the following snippet to `include/asm-x86_64/unistd.h` for each syscall:

```c
#define __NR_mysyscall ^^^ //(where ^^^ is one more than the highest number in the list).
__SYSCALL(__NR_mysyscall, sys_mysyscall)
```

Alternatively, you can plan to call your syscalls by number (for example, `syscall(123, param1, param2);`), and omit the `__SYSCALL` macro above.

### Implement the System Calls 

The default logic for `myreceive` is to block until a message is received.  You can add the task to the message queue wait list on `myreceive` if no message is available, and release a waiting process (if there is one) when calling `mysend`.  Here, we describe the implementation of your new syscalls in detail.

#### Copying Between Userspace and Kernel Space

Before we begin, note that the `mysend` and `myreceive` `buf` pointers are addresses that exist in user-space, not in kernel space!  The `__user` tags in the function prototypes indicate that the pointer is to a userspace address, and must be brought to/from the kernel using `copy_to_user` and `copy_from_user`.  You can [`kmalloc`](https://litux.nl/mirror/kerneldevelopment/0672327201/ch11lev1sec4.html) space in kernel memory to copy data between userspace virtual addresses and physical addresses for access by the kernel.  You can then put that kernel physical pointer onto the message queue when you send, and move that data from this kernel message queue back to user space on receive.  Read [this article](https://wiki.tldp.org/static/kernel_user_space_howto.html#Implementation-8) for details on how to use these functions, or see the example below taken from that article.  **Note that this is a model template from which you can write your two system calls, but it is intended to demonstrate the `copy_to_user` (and, analogously, the `copy_from_user`) functions.  It is not an exact copy of the code you will write.** 

```c
// From: https://wiki.tldp.org/static/kernel_user_space_howto.html#Implementation-8
// GNU Free Documentation License

#include <linux/linkage.h>
#include <asm/uaccess.h>
asmlinkage long sys_mysyscall(char __user *buf, int len)
{
    char msg [200]; // I'd malloc this and put the pointer in a task_struct queue for persistence!
    if(strlen_user(buf) > 200)
        return -EINVAL;
    copy_from_user(msg, buf, strlen_user(buf)); // copy the data to a kernel buffer
    printk("mysyscall: %s\n", msg);
    copy_to_user(buf, "hello world", strlen("hello world")+1); // put "hello world" back into their user buffer
    return strlen("hello world")+1;
}      
```

#### Implementing `mysend`

1. `kmalloc` a message structure to be added to some process' task_struct.  This is kernel memory, so you will pass `GFP_KERNEL` as the second parameter to `kmalloc`.  Don't roget to malloc enough room to fill the `char* msg` inside the message struct as well.  Set the other fields of the message.  

2. Find the `task_struct` corresponding to your target `pid`, and use `list_add_tail` to pass the address of the `list_head` in your struct and the address of the `list_head` in the target process message queue (in their `task_struct`).  

3. Finally, use `wake_up_interruptible` with the address of the `wait_queue_head_t` from the target process' message queue (in their `task_struct`) to wake up a process (if one exists) that may be waiting on your message.

#### Implementing `myreceive`

1. Begin by waiting for a message using `wait_event_interruptible`.  This function takes a `wait_queue_head_t`, and a condition to wait upon.  In this case, the condition is to wait until the list is not empty.  You can use the `!list_empty` function, which takes a `list_head`.  Take these from the current process.  

2. Iterate over the list using `list_for_each_entry_safe`, which is a macro that expands to a `for` loop, and takes as parameters two pointers to your `message` struct (the first one is the one you will use, and the second one is for temporary use by the macro), the address of your `list_head` message queue, and the word `list`.  

3. If there is a message on the linked list, copy its contents to your userspace using `copy_to_user` (which takes the `__user buf` you are passed via the syscall, the kernel buffer you are copying, and the number of bytes to copy (which you kept in your `message` structure during `mysend`!).  

4. Finally, use `list_del` to remove the item from the list (this takes the address of the `list_head` of the message structure you're removing).  

Once a message is found, your syscall has finished and you can break out of the loop.  
    
#### Thread-safe linked list operations

Manipulating kernel linked lists may not be thread-safe.  You can use `rcu_read_lock()` and `rcu_read_unlock()` to get atomic access to those structures.  Try to use these only when necessary (i.e., not for the entire function!).  `list_for_each_safe` is thread-safe by itself.
    
#### Don't forget to be robust

If you call `kmalloc`, check that the return value is not `NULL`.  If any call returns an error state like `0` or `-1` as appropriate for the call, clean up your memory with `kfree` and exit gracefully by returning a negative number as well.  Either way, be sure to `kfree` everything you `kmalloc` when you are able to do so.

### Build Wrapper Functions and a User Test Program

As in the [System Calls project](SyscallProcess), do not scatter raw `syscall(...)` calls through your test code. Write a small **wrapper function library** so your **user test program** reads like ordinary C. Real IPC libraries do exactly this — `msgsnd`/`msgrcv` are wrapper functions over syscalls.

```c
/* mailbox.h */
#ifndef MAILBOX_H
#define MAILBOX_H
#include <sys/types.h>
int mailbox_send(pid_t pid, int n, const char *buf);
long mailbox_receive(int n, char *buf);
#endif
```

```c
/* mailbox.c */
#include <unistd.h>
#include <sys/syscall.h>
#include "mailbox.h"

/* Replace the numbers with the syscall numbers you registered in unistd.h */
int  mailbox_send(pid_t pid, int n, const char *buf) { return syscall(288, pid, n, buf); }
long mailbox_receive(int n, char *buf)               { return syscall(289, n, buf); }
```

Now write a **user test program** that `fork`s a child and uses the wrapper functions:

1. Write a user test program that `fork`s a child.
2. The child creates a buffer and calls `mailbox_receive(...)` with that buffer, prints the message it receives, and `exit`s.
3. The parent calls `mailbox_send(...)` to send a message to the child by its `pid`, then `wait`s for the child before terminating.

Compile the wrapper library together with your test program:

```
gcc -o testmailbox testmailbox.c mailbox.c
```

## Getting Started

1. Boot your custom kernel (see [Booting a Custom Linux Kernel](BootingCustomKernel)) and make sure the tutorial `mygetpid` syscall from the [System Calls project](SyscallProcess) works. That confirms your build/register/boot loop is sound before you add the more complex mailbox syscalls.
2. Add the `mymessage` and `message_queue` structures to `sched.h`, then initialize the queue in `do_fork` and rebuild. Boot and confirm the kernel still runs normally — this checkpoint catches struct/initialization mistakes early.
3. Implement `mysend` first and verify (with a `printk`) that a message lands on the target's queue. **Expected checkpoint output:** a `dmesg` line showing the sender pid and message each time you send.
4. Implement `myreceive` and test the full `fork` parent/child round-trip. **Expected output:** the child prints exactly the message the parent sent.

## Common Pitfalls

* **Holding a spinlock across `copy_to_user` or `kmalloc(..., GFP_KERNEL)`.** Both can sleep; do the pointer surgery inside the lock and the copying/allocation outside it. See [kernel locking](../KernelDataStructures#part-2-kernel-locking).
* **Forgetting to `INIT_LIST_HEAD`** the queue in `do_fork`, so the first `list_add_tail` dereferences garbage.
* **`kfree`ing the wrong pointer** — free the `mymessage`, and remember to also free its `msg` buffer.
* **Waking the receiver inside the lock.** Call `wake_up_interruptible` after unlocking.
* **Copying past the buffer.** Copy only the `size` you stored during `mysend`, never more than `n`.

## Makefile Requirements

Include a `Makefile` that builds your user-space wrapper library and test program. It must support the following standard targets:

| Target       | What it must do                                                                     |
| ------------ | ----------------------------------------------------------------------------------- |
| `make`       | Compile `mailbox.c` together with your user test program(s), with no errors.        |
| `make run`   | Build if needed and launch the parent/child mailbox demo.                            |
| `make test`  | Build if needed and run your test cases, printing observable pass/fail output.       |
| `make clean` | Remove all executables, object files, and core dumps so a fresh `make` starts clean. |

## Glossary

* **Interprocess communication (IPC)** — mechanisms that let separate processes exchange data; here, a mailbox.
* **Wrapper function** — a user-space function (e.g., `mailbox_send`) that hides a syscall number behind a friendly signature by calling `syscall()`.
* **User test program** — a user-space program that calls your wrapper functions to exercise the mailbox syscalls.
* **`wait_queue_head_t`** — a kernel structure a task blocks on until an event (a message arriving) wakes it.
* **`copy_to_user` / `copy_from_user`** — the only safe way to move bytes across the user/kernel memory boundary.
* **Intrusive linked list** — the kernel's `struct list_head`, embedded inside your `mymessage`; see the [reference](../KernelDataStructures).
* **Blocking call** — a call that puts the caller to sleep until its condition is satisfied (here, `myreceive` until a message arrives).
