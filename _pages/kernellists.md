---
layout: default
permalink: /KernelDataStructures
title: "CS376: Operating Systems - Kernel Linked Lists, Locking, and File Data Structures"

---

# Kernel Data Structures: Linked Lists, Locking, and File Structures

## Why This Matters

When you write a system call for the [Mailbox](Projects/Mailbox) or [System Calls and Processes](Projects/SyscallProcess) projects, you are writing code that runs *inside* the kernel. You cannot `malloc`, you cannot call `printf`, and you cannot use the friendly `struct ListNode` you built in the [Memory Debugging lab](Labs/MemoryDebugC). Instead, the kernel gives you its own toolbox: an intrusive linked list (`struct list_head`), its own memory allocator (`kmalloc`), and its own family of locks. This page is a step-by-step reference for that toolbox so that the project pages can stay focused on *what* to build while this page explains *how* the underlying machinery works.

By the end of this page you should be able to:

* Read and write code that uses `struct list_head`, `LIST_HEAD`, `list_add`, `list_del`, and `list_for_each_entry`.
* Explain the `container_of` trick in your own words and trace it on a concrete example.
* Choose between a spinlock and a semaphore/mutex, and explain why interrupt context changes that choice.
* Describe what the kernel does when a user process calls `open()` and then `read()`.

---

## Part 1: The Kernel Linked List (`struct list_head`)

### The Big Idea: Intrusive Lists

In your C course you probably wrote a linked list like this, where each node *owns* its data:

```c
struct ListNode {
    int data;              // the payload
    struct ListNode* next; // the link
};
```

This is a **container-owns-link** design: the node holds the data. The Linux kernel does the *opposite*. It defines one tiny, reusable link structure and asks you to **embed it inside your own struct**. This is called an **intrusive** (or **embedded**) linked list:

```c
struct list_head {
    struct list_head *next, *prev;
};
```

That is the entire definition (from `include/linux/list.h`). Notice there is **no data field at all** — a `list_head` only knows how to point at other `list_head`s. To make a list of *your* data, you embed a `list_head` inside your struct:

```c
struct mymessage {
    pid_t sender_pid;
    size_t size;
    char* msg;
    struct list_head list;   // <-- the embedded link
};
```

### Text + Visual: Two Ways to Picture It

| Design            | Who owns the link?      | How you get from a link to the data      |
| ----------------- | ----------------------- | ---------------------------------------- |
| Classic (CS intro)| The data struct         | Follow `node->next`, read `node->data`   |
| Kernel (intrusive)| A shared `list_head`    | Follow `link->next`, then `container_of` back to the data |

Here is the same idea as an ASCII diagram. The list is **circular** and **doubly linked**, and the "head" is a sentinel node that holds no real message:

```
        +---------------------- HEAD (sentinel) ----------------------+
        |                                                             |
        v                                                             |
   [ list_head ] <--prev-- [ mymessage A ] <--prev-- [ mymessage B ]  |
     next -->  \             ^  .list                   ^  .list      |
               \            /                          /             |
                +----next--+-------------next---------+              |
                                                       \             |
                                                        +-----next---+
```

Every arrow points at a `list_head`, never directly at a `mymessage`. The magic of getting back to the `mymessage` is `container_of`, explained below.

### The Core Operations

You will only need a handful of macros and functions. Each is a one-liner in your code.

| Operation                         | What it does                                                            |
| --------------------------------- | ----------------------------------------------------------------------- |
| `LIST_HEAD(name);`                | Declares **and** initializes an empty list head named `name`.           |
| `INIT_LIST_HEAD(&head);`          | Initializes a `list_head` you already allocated (e.g., inside a struct).|
| `list_add(&new->list, &head);`    | Inserts `new` right **after** the head (stack / LIFO behavior).         |
| `list_add_tail(&new->list, &head);`| Inserts `new` at the **end** (queue / FIFO behavior).                   |
| `list_del(&node->list);`          | Unlinks `node` from whatever list it is on.                             |
| `list_empty(&head)`               | Returns true if the list has no entries.                                |
| `list_for_each_entry(pos, &head, member)` | Loops over every entry, giving you a typed pointer `pos`.       |
| `list_for_each_entry_safe(pos, n, &head, member)` | Same, but safe to `list_del` during the loop.           |

### `container_of`: Getting From a Link Back to Your Data

This is the one piece that confuses everyone the first time, so let us go slowly.

When you walk the list you have a `struct list_head *` in your hand. But you want the whole `struct mymessage` that it lives inside. How do you get there? The link is at a **known, fixed offset** inside your struct. If you know the address of the link, and you know how far the link sits from the top of the struct, you can subtract that distance and land on the top of the struct.

`container_of` does exactly that subtraction:

```c
container_of(ptr, type, member)
// ptr    = the address of the embedded member (your list_head*)
// type   = the struct type you want back (struct mymessage)
// member = the name of the embedded field (list)
```

**Worked example.** Suppose `struct mymessage` lays out in memory like this:

```
offset 0   : pid_t  sender_pid   (4 bytes)
offset 8   : size_t size         (8 bytes)
offset 16  : char*  msg          (8 bytes)
offset 24  : struct list_head list   <-- the embedded link lives here
```

The `list` member sits **24 bytes** from the start of the struct. Now suppose while walking the list we hold `ptr = 0x1018` (that is the address of some node's `.list` field). To recover the `mymessage`:

1. Start with the link address: `0x1018`.
2. Subtract the offset of `list` inside `mymessage`: `0x1018 - 24 = 0x1000`.
3. `0x1000` is the address of the whole `struct mymessage`. Done.

That is all `container_of` is: `(type*)((char*)ptr - offsetof(type, member))`. You almost never call it directly, because `list_for_each_entry` calls it for you.

### Putting It Together: A Worked Message-Queue Example

This is the pattern you will adapt for the [Mailbox project](Projects/Mailbox). Read it top to bottom; every line maps to a macro from the table above.

```c
// 1. Define your data with an embedded link (as above)
struct mymessage {
    pid_t sender_pid;
    size_t size;
    char* msg;
    struct list_head list;
};

// 2. Give each process a queue head (embedded in task_struct, initialized in do_fork)
struct message_queue {
    struct list_head messages;    // the sentinel head
    wait_queue_head_t wait;       // to block on when empty (see Part 2)
};

// ENQUEUE (inside mysend): add a message to the tail so oldest is read first
void enqueue(struct message_queue *q, struct mymessage *m)
{
    list_add_tail(&m->list, &q->messages);   // FIFO
}

// DEQUEUE (inside myreceive): walk the list, take the first entry
struct mymessage *dequeue(struct message_queue *q)
{
    struct mymessage *m, *tmp;
    // list_for_each_entry_safe gives us 'm' as a real struct mymessage*
    // (it called container_of for us!) and 'tmp' so we can delete safely.
    list_for_each_entry_safe(m, tmp, &q->messages, list) {
        list_del(&m->list);   // unlink this node
        return m;             // caller now owns it and must kfree it
    }
    return NULL;              // list was empty
}
```

Notice you never wrote `container_of` yourself and you never touched a raw `next`/`prev` pointer. That is the payoff of the intrusive design: one list implementation works for *any* struct, with no casts to `void*` and no separate node allocations.

### Common Pitfalls

* **Forgetting to initialize the head.** A `list_head` embedded in a struct must be initialized with `INIT_LIST_HEAD(&q->messages)` (the Mailbox project does this in `do_fork`). An uninitialized head has garbage `next`/`prev` and will crash on the first `list_add`.
* **Using `list_for_each_entry` while deleting.** If you `list_del` a node during a plain `list_for_each_entry`, the loop follows a freed `next` pointer. Use the `_safe` variant, which stashes the next pointer before your loop body runs.
* **Confusing the head with a data node.** The head is a sentinel; it is *not* a `mymessage`. Never call `container_of` on the head.
* **Freeing the link instead of the container.** Call `kfree` on the `mymessage`, not on `&m->list` (they are the same address only when `list` is the first field — do not rely on that).

---

## Part 2: Kernel Locking

### Why This Matters

Your linked-list code above has a hidden danger: two CPUs (or an interrupt and your syscall) might touch the same message queue at the same time. If one thread is halfway through `list_add_tail` when another starts `list_del`, the `next`/`prev` pointers become inconsistent and the kernel corrupts memory. You must **serialize** access to shared kernel data structures with a lock. The kernel gives you two main families, and choosing wrong can **hang the whole machine**.

### Spinlocks vs. Semaphores/Mutexes

| Feature                    | Spinlock                                   | Semaphore / Mutex                          |
| -------------------------- | ------------------------------------------ | ------------------------------------------ |
| While waiting, the CPU...  | **spins** (busy-waits), doing no other work| **sleeps**; the scheduler runs other tasks |
| Safe to hold while sleeping?| **No** — never block/sleep while holding one| Yes                                        |
| Usable in interrupt context?| **Yes** (this is the main reason it exists)| **No** — interrupts cannot sleep           |
| Best for...                | very short critical sections               | longer critical sections that may sleep     |
| Cost of waiting            | wastes CPU cycles, but no context switch   | context-switch overhead, but CPU stays busy |

The rule of thumb: **if the critical section is short and you might be in interrupt context, use a spinlock; if it is longer or you must sleep (e.g., call `copy_to_user`, `kmalloc(..., GFP_KERNEL)`, or wait on a queue), use a semaphore or mutex.**

### The Interrupt-Context Rule (the part people get wrong)

An **interrupt handler** cannot go to sleep — there is no process context to put to sleep and later wake. This has two consequences:

1. Inside an interrupt handler you may **only** use spinlocks, never semaphores/mutexes.
2. If a lock is *also* taken by an interrupt handler, then code in normal (process) context must disable interrupts while holding it — otherwise the interrupt could fire on the same CPU and spin forever waiting for a lock the interrupted code already holds. That is a self-**deadlock**. Use `spin_lock_irqsave(&lock, flags)` / `spin_unlock_irqrestore(&lock, flags)` for this case.

### Worked Examples

**Spinlock protecting a short list update:**

```c
#include <linux/spinlock.h>

spinlock_t queue_lock;
spin_lock_init(&queue_lock);   // once, at init time

// short critical section: just pointer surgery, no sleeping
spin_lock(&queue_lock);
list_add_tail(&m->list, &q->messages);
spin_unlock(&queue_lock);
```

**Spinlock that an interrupt also uses (disable interrupts):**

```c
unsigned long flags;
spin_lock_irqsave(&queue_lock, flags);   // save + disable IRQs on this CPU
list_del(&m->list);
spin_unlock_irqrestore(&queue_lock, flags);  // restore previous IRQ state
```

**Semaphore/mutex for a section that may sleep** (for example, because it copies to user space):

```c
#include <linux/mutex.h>

struct mutex mbox_lock;
mutex_init(&mbox_lock);   // once, at init time

mutex_lock(&mbox_lock);            // may sleep while waiting — that's fine here
copy_to_user(buf, m->msg, m->size);// copy_to_user itself may sleep
mutex_unlock(&mbox_lock);
```

> **Note on this old kernel.** The 2.6.22 kernel used in our projects still exposes the coarse `lock_kernel()` / `unlock_kernel()` "Big Kernel Lock" (BKL), which the [System Calls project](Projects/SyscallProcess) uses to keep a task from exiting during `myjoin`. The BKL is a sleeping lock that serializes large regions and has since been removed from Linux. Prefer a fine-grained spinlock or mutex around *just* the data you are protecting whenever the project allows it; reach for the BKL only where the project instructions explicitly call for it.

### Locking + Linked Lists Together

The two halves of this page combine directly. Whenever you touch a shared `list_head`, wrap the operation in a lock:

```c
// SEND side
spin_lock(&q->lock);
list_add_tail(&m->list, &q->messages);
spin_unlock(&q->lock);
wake_up_interruptible(&q->wait);   // wake a blocked receiver (do this OUTSIDE the spinlock)

// RECEIVE side
spin_lock(&q->lock);
if (!list_empty(&q->messages)) {
    m = list_first_entry(&q->messages, struct mymessage, list);
    list_del(&m->list);
}
spin_unlock(&q->lock);
// now, safely OUTSIDE the lock (because it may sleep):
if (m) copy_to_user(buf, m->msg, m->size);
```

Note the discipline: **do the pointer surgery inside the lock, but do the sleeping work (`copy_to_user`, `wake_up`) outside it.** Holding a spinlock across a call that can sleep is a classic kernel bug. The Mailbox project's mention of `rcu_read_lock()`/`rcu_read_unlock()` is a lighter-weight alternative for read-mostly traversal; use whichever the project specifies, but keep the "no sleeping inside a spinlock" rule in mind.

### Common Pitfalls

* **Sleeping while holding a spinlock** (calling `kmalloc(..., GFP_KERNEL)`, `copy_to_user`, or `mutex_lock`). This can deadlock the machine. `kmalloc` outside the lock, or use `GFP_ATOMIC` if you truly must allocate inside.
* **Forgetting `_irqsave`** on a lock shared with an interrupt handler — an intermittent, hard-to-reproduce hang.
* **Locking too much.** Holding one giant lock for the whole syscall serializes everything and kills performance. Lock only the shared structure, for as short a time as possible.
* **Waking up inside the lock.** `wake_up_interruptible` can schedule; do it after `spin_unlock`.

---

## Part 3: Reading File Data Structures

### Why This Matters

The [File I/O assignment](Assignments/FileIO) and [File I/O lab](Labs/FileIO) have you call `open()`, `read()`, and `close()`. Those three lines set off a chain of events across four kernel data structures. Understanding that chain explains *why* a file descriptor is "just an int," why two processes can share an open file, and where the file's current position actually lives.

### The Four Structures

| Structure         | One per...                    | Holds                                                   |
| ----------------- | ----------------------------- | ------------------------------------------------------- |
| **fd table**      | process (in `task_struct`)    | an array mapping small ints (`0,1,2,...`) to `struct file*` |
| **`struct file`** | each `open()` call            | current offset (`f_pos`), access mode, pointer to the `dentry`/`inode` |
| **`struct dentry`**| each path component in cache  | a name (`"hosts"`) and a link to its `inode` and parent |
| **`struct inode`**| each file **on disk**         | size, permissions, owner, and where the data blocks live|

The key insight: **the file descriptor `int` is just an index** into your process's fd table. It is not the file. Following the arrows: `fd (int)` → `task_struct.files[fd]` → `struct file` → `struct dentry` → `struct inode` → the actual bytes on disk.

```
  Process A                          Shared kernel objects
  ---------                          ---------------------
  fd table
  [0] stdin
  [1] stdout
  [2] stderr
  [3] ---------> struct file  ---> struct dentry ---> struct inode ---> data blocks
                 (offset=0,          ("hosts")         (size, perms,      on disk
                  mode=O_RDONLY)                        block map)
```

### What Happens on `open("hosts", O_RDONLY)` — Step by Step

1. The C library issues the `open` **system call**, trapping into the kernel.
2. The kernel does a **path lookup**: it walks the path one component at a time, consulting the **dentry cache** (`dcache`) for each name. A `dentry` links a name to an `inode`.
3. When it reaches the final component, it has the file's **`inode`** — the on-disk record of the file's size, permissions, owner, and block locations. Permissions are checked here (this is exactly the check `forcewrite` skips in the [Syscall project](Projects/SyscallProcess)).
4. The kernel allocates a new **`struct file`**, sets its offset `f_pos = 0` and its mode to `O_RDONLY`, and points it at the dentry/inode.
5. The kernel finds the **lowest free slot** in the process's **fd table**, stores the `struct file*` there, and returns that slot number to the user. That integer is your file descriptor.

### What Happens on `read(fd, buf, n)` — Step by Step

1. `fd` indexes the process's fd table to find the `struct file*`.
2. The kernel reads starting at the file's current offset `f_pos`, pulling data from the pages backing the `inode` (from the page cache, or from disk on a miss).
3. It **`copy_to_user`s** the bytes into your `buf` (the same user/kernel boundary you cross in the Mailbox project).
4. It **advances `f_pos`** by the number of bytes read, so the next `read` continues where this one left off. This is why the offset lives in `struct file` and not in the `inode`: two independent `open()`s of the same file get two `struct file`s with two independent offsets, but share one `inode`.

### Worked Example: Why `fork()` Shares an Offset but a Second `open()` Does Not

* After `fork()`, parent and child share the **same `struct file`** (the fd table entry is copied, but it points at the same object). If the child reads 100 bytes, the parent's next read starts at offset 100 — they share `f_pos`.
* If instead each process calls `open()` separately, each gets its **own `struct file`** with its **own `f_pos`**, but both point at the **same `inode`**. Reads are independent.

This distinction is a favorite exam question and explains a lot of "why did my file position jump?" bugs.

### Common Pitfalls

* **Thinking the offset lives in the inode.** It lives in `struct file`. The inode is shared; the offset is per-open.
* **Assuming `fd` numbers are global.** They are per-process indices. `fd 3` in two processes are unrelated unless inherited via `fork`.
* **Forgetting `close()` frees the `struct file`, not the inode.** The inode persists as long as the file exists on disk (and is cached).

---

## Glossary

* **Intrusive list** — a linked list where the link (`list_head`) is embedded inside your data struct rather than owning the data.
* **`container_of`** — a macro that recovers a pointer to the enclosing struct given a pointer to one of its embedded members, by subtracting the member's offset.
* **Sentinel head** — a `list_head` that marks the start of a circular list but holds no real payload.
* **Critical section** — a region of code that touches shared data and must not run concurrently on two CPUs.
* **Spinlock** — a lock whose waiter busy-waits (spins); safe in interrupt context, unsafe to sleep while holding.
* **Semaphore / mutex** — a sleeping lock; the waiter is put to sleep and the CPU runs other work. Not usable in interrupt context.
* **Interrupt context** — code running in response to a hardware interrupt; cannot sleep, so cannot use sleeping locks.
* **File descriptor** — a small non-negative integer that indexes a process's file descriptor table.
* **inode** — the on-disk (and in-memory) record describing a file: size, permissions, owner, and data block locations.
* **dentry** — a directory-entry cache object linking a path-component name to an inode.

## See Also

* [Mailbox Project](Projects/Mailbox) — apply the kernel linked list and locking material here.
* [System Calls and Processes Project](Projects/SyscallProcess) — task list traversal and `lock_kernel()`.
* [File I/O Assignment](Assignments/FileIO) and [File I/O Lab](Labs/FileIO) — the user-space side of the file structures above.
* [GDB and Valgrind](GDBValgrind) — debugging tools, including kernel debugging with QEMU + gdb.
