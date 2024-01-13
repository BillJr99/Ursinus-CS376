---
layout: assignment
permalink: Assignments/Threads
title: "CS376: Operating Systems - Threaded Programming"
excerpt: "CS376: Operating Systems - Threaded Programming"

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

Implement **two** versions of the office hours problem, first using semaphores, and then using condition variables.  Use print statements to show the behavior of the threads as they execute.

Run your program using the `valgrind` thread checker, and provide the report.

## Part 3: Makefile

Write a makefile to compile and run your programs.  You may write separate makefiles for each program, if you prefer, but each should have a target to build the program, and one to run/test the program.