---
layout: assignment
permalink: Assignments/Threads
title: "CS376: Operating Systems - Threaded Programming"

info:
  coursenum: CS376
  points: 100
  goals:
    - Design and implement a C program to simulate multiple threads
    - Use mutex locks to protect shared variables
    - Investigate the implications of race conditions in concurrent programming
    - Evaluate the performance impact of different synchronization techniques
    - Implement correct concurrent programs that coordinate using semaphores and condition variables
        
  rubric:
    - weight: 20
      description: Program Correctness
      preemerging: The program does not compile or runs with significant errors.
      beginning: The program compiles but does not function as expected, showing incorrect behavior or outputs.
      progressing: The program runs with minor issues; some elements of thread coordination or synchronization are not implemented correctly.
      proficient: The program runs correctly, handling thread synchronization effectively and producing accurate outputs as per the assignment requirements.
    - weight: 20
      description: Proper Thread Coordination
      preemerging: Thread coordination is absent or implemented incorrectly, leading to severe race conditions or deadlocks.
      beginning: Basic thread coordination is implemented, but it's prone to occasional race conditions or inefficiencies.
      progressing: Thread coordination is mostly correct, with minor issues in handling synchronization or resource sharing.
      proficient: Excellent implementation of thread coordination, demonstrating a strong understanding of synchronization mechanisms, preventing race conditions and deadlocks.
    - weight: 20
      description: Answering Assignment Questions
      preemerging: Responses to the assignment questions are missing or largely incorrect.
      beginning: Partially correct responses to the assignment questions, but lacking in detail or accuracy.
      progressing: Responses address most of the assignment questions accurately, with some gaps in thoroughness or detail.
      proficient: Comprehensive and accurate responses to all assignment questions, demonstrating deep understanding and analysis.
    - weight: 10
      description: Inclusion of Valgrind Report
      preemerging: No Valgrind report is included in the submission.
      beginning: A Valgrind report is included but is incomplete or shows significant memory leaks or errors.
      progressing: The Valgrind report is complete, showing minor memory issues or warnings.
      proficient: A comprehensive Valgrind report is included, demonstrating effective memory management with no leaks or errors.
    - weight: 15
      description: Writing a README
      preemerging: README is absent or lacks basic instructions on how to run the program.
      beginning: README includes some instructions, but they are unclear or incomplete.
      progressing: README is well-written with clear instructions on running the program, but lacks some details or explanations.
      proficient: A comprehensive README is provided, with detailed, clear instructions on how to run the program and explanations of its functionality.
    - weight: 15
      description: Providing a Makefile
      preemerging: No Makefile is provided, or it fails to compile the program.
      beginning: Makefile is provided but has errors or lacks functionality for easy compilation.
      progressing: Makefile compiles the program correctly but lacks advanced features or is not fully optimized.
      proficient: An excellent Makefile is provided, facilitating easy compilation and offering features such as clean, build, and test options.

  readings:
    - rlink: https://makefiletutorial.com/
      rtitle: Makefile Tutorial
      
tags:
  - concurrent

---

## Part 1: Introduction to Concurrent Programming

Write a program in C to simulate 100 people (threads) that enter a stadium.  They get up a few times, let's say 1000 times (for food, etc.), and return to their seats.  They pass through a counter each time they return to their seat.  Write a program that consists of one threaded function that you invoke via 100 pthreads.  That threaded function should increment a shared variable (initialized to 0, and declared as a volatile variable), 1000 times each.  Note - if your system does not allow you to spawn so many threads, you may use fewer threads and/or a larger number of "times they get up."

Be sure to declare this variable `volatile`, and describe in your writeup why this is important.

Run this program 10 times, and time the runs.  What is the average runtime of these runs?  Also, record the final output of the counter.  It should be 100000, but it likely will not be.  What was it on each run?

### Correcting the Race Condition with Mutex Locks

What is the problem with this program?  Use the `valgrind` thread checker to identify the program's thread failures, and provide the report showing the problems it identified.

Now, create a new version of your program, and using `mutex` locks (lock inside the loop), fix the code, and re-run it 10 times.  Again, time the runs, and give the average runtime.  As before, give the final output of the counter (this time, it really should be 100000).  How much slower was this program than the one without mutex locks? 

Now, in a new version of the program, move the locks so that they lock and unlock only once per thread (outside the loop).  Verify the correct value is still obtained, and find the average runtime over 10 runs again.  What do you observe as compared to the other two versions?

## Part 2: The Office Hours Problem

Consider the following problem: your professor has a small office that can seat three students simultaneously.  During office hours, students arrive after a random duration (perhaps after "`sleep`ing" for a random amount of time!) to the professor's office to ask questions and have disucssion.  Professors do no other work during their office hours, so they simply block if no students are there and until one arrives.  Once the professor is working with three students simultaneously, additional students will queue up outside by blocking; they are awoken when a student leaves the office.

Using mutex locks, implement a solution to this problem.  To do this, create a shared `int` variable representing a counter of the number of students waiting for office hours.  This variable is initially `0` and should be protected by a mutex lock (also accessible to all threads, just like the shared counter).

The threaded function should check the value of this counter.  If it is less than `3`, increment the variable and enter the office (by proceeding through the function).  Decrement the variable right before you return, simulating leaving the office.  Don't forget to lock your mutex when accessing this variable!

If the shared counter is greater than or equal to 3, `malloc` a new lock and insert it at the head of a linked list.  Increment a new shared variable representing the number of students in the waiting room (also known as the size of the linked list!).  Again, don't forget to protect this access with a mutex!  Lock the lock twice (once to lock it, and once more to block on it -- since you want to wait outside the office for a seat to open up!).  Hint: protect adding the node to the linked list with your mutex, but be sure to unlock the mutex right before locking your new waiting room lock the second time (why?).

When a student leaves the office, check the size of the waiting room queue.  If it is greater than `0`, remove the tail of the linked list (the student who was waiting the longest), and decrement the waiting room size (again, all protected by a mutex!).  Unlock the lock you just popped off of the linked list to "wake up" the student in the waiting room.  Be sure to destroy this lock so that you free the resources it was using.

Run your program using the [`valgrind` thread checker](https://valgrind.org/docs/manual/hg-manual.html); specifically, by running `valgrind --tool=helgrind ./a.out`, and provide the report.  Be sure to destroy all your mutexes at the end of your program and free your dynamically allocated memory.  Use the `valgrind` leak check to verify you have no memory leaks, and include this report in your submission. 

### What to Do

To solve this problem, it is not necessary to implement a professor thread (only a student thread).  When a student needs to wait for the professor (i.e., `numstudents >= 3`), you can create a lock for the student thread to block on.  To do this, when it is time for a thread to block and wait, you'll:

1. `malloc` a `pthread_mutex_t`
2. Lock the mutex once (so it's locked)
3. Add that mutex to the linked list
4. Lock it again (so your thread blocks on it).  

When a student leaves the office, they can check if the linked list is not `NULL`.  If it's not `NULL`, then remove a lock off of the linked list, and unlock it (which releases the next student waiting outside to come in).

## Part 3: Makefile

Write a makefile to compile and run your programs.  You may write separate makefiles for each program, if you prefer, but each should have a target to build the program, and one to run/test the program.