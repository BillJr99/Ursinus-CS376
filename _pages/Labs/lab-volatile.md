---
layout: assignment
permalink: Labs/Volatile
title: "CS376: Operating Systems - Volatile Variables and Thread Visibility"

info:
  coursenum: CS376
  points: 100
  goals:
    - Diagnose a concurrency bug by starting from non-working code and observing its failure
    - Distinguish visibility, atomicity, and instruction reordering as three separate problems
    - Explain what the volatile keyword does and, crucially, what it does not do
    - Apply mutex locks to achieve correct synchronization where volatile alone fails

  rubric:
    - weight: 25
      description: Reproducing and Documenting the Bug
      preemerging: The broken program is not run, or no observations of its incorrect behavior are recorded.
      beginning: The broken program is run, but observations are vague or the incorrect output is not documented across multiple runs.
      progressing: The broken program is run multiple times with recorded (and varying) incorrect counter values, but the observations lack analysis.
      proficient: The broken program is run many times with a clear table of varying incorrect results, plus a helgrind report that documents the data race.
    - weight: 25
      description: Explaining Visibility, Atomicity, and Reordering
      preemerging: The three concepts are not addressed or are conflated with one another.
      beginning: At least one concept is described, but with significant inaccuracy.
      progressing: All three concepts are described correctly but with limited connection to the observed program behavior.
      proficient: All three concepts are explained accurately, distinguished from one another, and tied directly to the program's observed failures.
    - weight: 25
      description: Correct Synchronized Solution
      preemerging: No working solution is provided, or it still produces incorrect results.
      beginning: A solution is attempted with a mutex but it deadlocks, leaks, or still races.
      progressing: A mutex-based solution produces the correct count but has minor issues (e.g., missing destroy, over-locking).
      proficient: A mutex-based solution reliably produces the correct count with no leaks and a clean helgrind report, and correctly explains why volatile alone was insufficient.
    - weight: 15
      description: Writeup and README
      preemerging: An incomplete submission is provided.
      beginning: The program is submitted but is missing the required writeup or answers.
      progressing: A README is provided describing the solution and answering nearly all questions.
      proficient: A README is provided that describes the solution and answers all questions posed in the instructions.
    - weight: 10
      description: Makefile
      preemerging: No Makefile is provided, or it fails to build the programs.
      beginning: A Makefile is provided but is missing required targets.
      progressing: A Makefile builds the programs and runs at least one of the required targets.
      proficient: A Makefile provides make, make run, make test, and make clean, each behaving as described.

  readings:
    - rlink: https://computing.llnl.gov/tutorials/pthreads/
      rtitle: "POSIX Threads Programming (LLNL)"
    - rlink: https://valgrind.org/docs/manual/hg-manual.html
      rtitle: "Helgrind: a thread error detector"
    - rlink: ../GDBValgrind
      rtitle: "Helpful Debugging Tools: GDB and Valgrind"

tags:
  - concurrent
  - c
  - threads

---

> **Core Concepts — why this matters.** This lab reinforces the essential OS topics of *concurrency*, *shared-memory synchronization*, and *the memory model* that the [Threaded Programming assignment](../Assignments/Threads) builds on. Rather than starting from a blank file, you will start from a program that is **deliberately broken** and walk it through a scientific loop: *observe* the wrong answer, *form a hypothesis* about why, and *fix* it. Along the way you will learn the single most misunderstood keyword in concurrent C — `volatile` — and why it is **not** a synchronization tool.

## The Scenario

Two threads share one counter. Each thread increments it a million times. When both finish, the counter *should* read exactly two million. It almost never will. Your job is to find out why, and to fix it properly.

This lab is structured as three passes over the same tiny program:

1. **Pass 1 — Observe the bug** with a plain shared counter.
2. **Pass 2 — Try `volatile`** and discover it does *not* fix the count.
3. **Pass 3 — Fix it with a mutex** and explain why that was the piece that was actually missing.

## Getting Started

1. Create the file `counter.c` below exactly as written. Do **not** fix it yet — we want to see it fail.
2. Compile with threading enabled: `gcc -pthread -o counter counter.c`
3. Run it several times: `./counter`
4. **Expected (broken) output:** a "Final count" that is *less than* 2000000 and *changes from run to run*. If you happen to get 2000000 once, run it again — the bug is intermittent by nature.

## Pass 1: The Broken Program

```c
/* counter.c -- BROKEN ON PURPOSE. Do not fix yet; run it and observe. */
#include <stdio.h>
#include <pthread.h>

#define ITERATIONS 1000000

int counter = 0;   /* shared between both threads */

void* increment(void* arg) {
    for (int i = 0; i < ITERATIONS; i++) {
        counter = counter + 1;   /* the dangerous line */
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("Final count: %d (expected %d)\n", counter, 2 * ITERATIONS);
    return 0;
}
```

### Observe

Run the program ten times and record the "Final count" each time in a small table like this:

| Run | Final count | Expected |
| --- | ----------- | -------- |
| 1   | 1421885     | 2000000  |
| 2   | 1638002     | 2000000  |
| ... | ...         | ...      |

(Your numbers will differ — that variability is the whole point.)

### Hypothesize

Why is the count too low, and why is it *different every time*? The culprit is the line `counter = counter + 1`. Although it is a single line of C, the CPU executes it as **three separate steps**:

```
1. LOAD  counter   -> register      (read the current value)
2. ADD   register, 1                (compute value + 1)
3. STORE register  -> counter       (write it back)
```

Now picture two threads interleaving these steps. This is called a **race condition**:

```
counter starts at 100

Thread A: LOAD counter (reads 100)
Thread B: LOAD counter (reads 100)     <-- both read the SAME old value
Thread A: ADD -> 101
Thread B: ADD -> 101
Thread A: STORE counter = 101
Thread B: STORE counter = 101          <-- one increment is LOST

counter is now 101, but TWO increments happened. It should be 102.
```

Every time two increments overlap like this, one is silently lost. The number of collisions is random, so the final count is random too. Confirm the data race with helgrind:

```
gcc -g -pthread -o counter counter.c
valgrind --tool=helgrind ./counter
```

Include helgrind's report of the race on `counter` in your writeup.

## Pass 2: Try `volatile` (and watch it fail to fix the count)

A very common — and wrong — first instinct is: "the compiler must be caching `counter` in a register; I'll mark it `volatile` to force it to memory." Try it. Change one line:

```c
volatile int counter = 0;   /* Pass 2: does this fix the count? */
```

Rebuild and run ten more times. **Record the results.** You will find the count is *still* wrong and *still* varies. `volatile` did **not** fix the bug.

### Hypothesize: the three separate problems

`volatile` did not help because the bug was never the problem that `volatile` solves. There are **three distinct concurrency problems**, and they are easy to confuse:

| Problem       | Question it answers                                          | Does `volatile` solve it? |
| ------------- | ----------------------------------------------------------- | ------------------------- |
| **Visibility**| "When one thread writes, will another thread *see* the new value?" | **No.** It only stops the *compiler* from caching the value in a register — it creates no happens-before relationship, so with a data race the other thread still has no guarantee of seeing the write. |
| **Atomicity** | "Does `counter = counter + 1` happen as one indivisible step?" | **No.** `volatile` does not make read-modify-write atomic. |
| **Reordering**| "Can the compiler/CPU move instructions around this access?"  | Only for the `volatile` access itself; it is **not** a general memory barrier and does not order other variables. |

Our bug is an **atomicity** failure: two threads perform the three-step load-add-store and clobber each other — and as the table shows, `volatile` solves none of the three problems for threads. This is the key lesson:

> **`volatile` is not a synchronization primitive.** It tells the compiler "this variable may change unexpectedly, so always read it from memory." It says **nothing** about making a compound operation atomic and it provides **no** mutual exclusion. In C, `volatile` is for things like memory-mapped hardware registers and signal handlers — *not* for coordinating threads.

## Pass 3: Fix It Properly with a Mutex

The missing ingredient is **mutual exclusion**: only one thread may execute the load-add-store at a time. A `pthread_mutex_t` provides exactly this. Note that we initialize it at runtime with `pthread_mutex_init` and clean it up with `pthread_mutex_destroy`.

```c
/* counter.c -- CORRECT version */
#include <stdio.h>
#include <pthread.h>

#define ITERATIONS 1000000

int counter = 0;               /* volatile is not needed once the mutex is correct */
pthread_mutex_t lock;          /* declare the lock */

void* increment(void* arg) {
    for (int i = 0; i < ITERATIONS; i++) {
        pthread_mutex_lock(&lock);     /* enter the critical section */
        counter = counter + 1;         /* now indivisible with respect to other threads */
        pthread_mutex_unlock(&lock);   /* leave the critical section */
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_mutex_init(&lock, NULL);   /* runtime initialization */

    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&lock);      /* release the lock's resources */
    printf("Final count: %d (expected %d)\n", counter, 2 * ITERATIONS);
    return 0;
}
```

### Observe

Run this version ten times. It should print **2000000 every single time**. Run it under helgrind again; the race report should be gone.

### Hypothesize / Reflect

Answer these in your README:

1. Why does the mutex fix the count when `volatile` did not? (Frame your answer in terms of atomicity vs. visibility.)
2. The mutex version is noticeably slower than the broken version. Time both with `time ./counter`. Where does the extra time go?
3. Once the mutex is in place, is `volatile` still needed on `counter`? Why or why not? (Hint: the lock/unlock functions already act as memory barriers that force visibility.)

## Common Pitfalls

* **Believing `volatile` makes `counter++` thread-safe.** It does not. This is the single most common misconception this lab exists to correct.
* **Forgetting `-pthread`** when compiling, causing link errors or silently broken threading.
* **Locking around the wrong region** — e.g., locking outside the loop entirely defeats the purpose only if you also read/modify outside it; locking *inside* the loop around the shared access is correct here.
* **Forgetting `pthread_mutex_destroy`**, which leaks the lock's resources (helgrind/memcheck will flag it).
* **Using `PTHREAD_MUTEX_INITIALIZER` when a runtime init is clearer.** Prefer `pthread_mutex_init(&lock, NULL)` so you can check for errors and pass attributes; see the [Threaded Programming assignment](../Assignments/Threads) for the full rationale.

## Makefile Requirements

Include a `Makefile` supporting the following standard targets:

| Target       | What it must do                                                                  |
| ------------ | -------------------------------------------------------------------------------- |
| `make`       | Compile the broken and corrected programs with `-pthread`, with no errors.       |
| `make run`   | Build if needed and run the corrected program, printing the final count.         |
| `make test`  | Build if needed and run both versions (and/or helgrind), showing pass/fail output.|
| `make clean` | Remove all executables, object files, and core dumps so a fresh `make` starts clean. |

## Glossary

* **Race condition** — a bug where the result depends on the unpredictable interleaving of threads.
* **Critical section** — a region of code that accesses shared data and must run without interruption by another thread.
* **Mutual exclusion (mutex)** — a lock ensuring at most one thread is in a critical section at a time.
* **Atomicity** — the property that an operation happens as one indivisible step; `counter = counter + 1` is *not* atomic.
* **Visibility** — whether a write by one thread becomes observable to another thread.
* **Instruction reordering** — the compiler's or CPU's freedom to execute instructions out of program order for performance.
* **`volatile`** — a C qualifier that forces reads/writes to go to memory; it addresses visibility only and provides **no** atomicity or mutual exclusion.
* **Memory barrier** — an operation (such as a mutex lock/unlock) that constrains reordering and forces visibility.
