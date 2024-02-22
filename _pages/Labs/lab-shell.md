---
layout: assignment
permalink: Labs/Shell
title: "CS376: Operating Systems - Shell"

info:
  coursenum: CS376
  points: 100
  goals:
    - To use an editor in the non-graphical Unix environment.
    - To execute standard Unix commands.
    - Understand process creation and control in a Unix-like environment.
    - Gain experience with signal handling and inter-process communication.
    - Develop a deeper understanding of system calls like fork(), exec(), and waitpid().

  rubric:
    - weight: 40
      description: Functionality - Ability to execute commands, handle redirection, and manage processes correctly.
      preemerging: Shell does not compile or run. 
      beginning: Shell compiles but has significant issues in executing most commands.
      progressing: Shell executes some commands but struggles with either redirection or process management. 
      proficient: Shell executes commands correctly, handles both input and output redirection well, and manages processes effectively. 
    - weight: 20
      description: Signal Handling - Correct implementation and handling of SIGINT and SIGCHLD.
      preemerging: No signal handling implemented. 
      beginning: Partial or incorrect implementation of one signal. 
      progressing: Handles both signals, but with issues in forwarding or reaping processes. 
      proficient: Correctly implements handling of both SIGINT and SIGCHLD, including forwarding and process reaping. 
    - weight: 10
      description: Makefile - Inclusion and functionality of a Makefile for compiling the shell.
      preemerging: No Makefile provided. 
      beginning: Makefile provided but with errors or missing targets. 
      progressing: Makefile works but lacks proper organization or documentation. 
      proficient: Makefile is well-organized, documented, and compiles the shell correctly. 
    - weight: 10
      description: Exploring the Unix Shell - Completion and understanding of tasks in the "Exploring the Unix Shell" section.
      preemerging: Tasks not attempted or largely incorrect. 
      beginning: Tasks attempted but with significant errors or misunderstandings. 
      progressing: Tasks mostly completed with minor errors or misunderstandings.
      proficient: Tasks completed correctly with a clear understanding demonstrated. 
    - weight: 10
      description: Code Quality - Organization, readability, and commenting of the code.
      preemerging: Code is disorganized, poorly commented, and difficult to read. 
      beginning: Code has some organization and comments but lacks clarity. 
      progressing: Code is mostly well-organized and commented, with some areas lacking clarity. 
      proficient: Code is well-organized, clearly written, and thoroughly commented. 
    - weight: 10
      description: Documentation - Quality of the report explaining the implementation, signal handling, and process management.
      preemerging: No report or largely off-topic. 
      beginning: Report provided but lacks detail or contains significant inaccuracies. 
      progressing: Report covers key aspects but lacks depth or clarity in some areas. 
      proficient: Comprehensive, clear, and accurate report of the implementation. 

  readings:
    - rlink: "../files/lectures/shellipc.pptx"
      rtitle: "Notes on IPC and the Shell"
      
tags:
  - shell

---

## Part 1: Exploring the UNIX Shell

* Create a subfolder in your home directory and populate it with a number of files (their names and contents do not matter).  Give them different file permissions (they don't matter -- just mix them up).  Make some directories as well.  

* Within this subdirectory:

  * Write a UNIX command to find all files that are files (that is, not directories -- and even files that are in subdirectories)
  * Write a UNIX command to find all directories (even subdirectories)
  * Write a UNIX command to change file permissions on all files to 600
  * Write a UNIX command to change all file permissions on all directories to 755

* Write a UNIX command to replace all instances of the word "foo" in a file with "bar"

* Use the UNIX pipe to connect the ls command and the sort command to print all of your files in alphabetical order.

* Use the UNIX pipe with the du and sort commands to list all files in your home directory, sorted from smallest file to largest file.  Be careful not to sort the sizes "alphabetically" but rather "numerically."

* Use the UNIX pipe with the ls and wc commands to print the number of files in your current working directory.

* The command `ps -auxw` prints all processes being executed by all users.  Use the `grep` and `cut` commands in conjunction with the `ps` command to print the `pid` number of all processes being executed by your user ID.

## Part 2: The vim Editor

Run and complete the `vimtutor` program to learn how to edit files through your console without a GUI.

## Part 3: Shell Architecture

You will create a simple shell program in C that allows the user to execute commands with options and redirect standard input/output. Your shell will need to handle signals appropriately, specifically `SIGINT` and `SIGCHLD`.  You do not need to support running processes in the background; after a command execution, your shell should wait until the child process has finished before returning to the prompt.

### Basic Shell Functionality
1. **Command Execution**: Your shell should allow the user to run commands with arguments. For example, `ls -l /home`.
2. **Input/Output Redirection**: Implement redirection of standard input and standard output. For instance, `cat < input.txt > output.txt`.  You do not need to support pipes.
3. **Prompt**: After executing a command or if no command is running, the shell should display a prompt and wait for user input.

### Signal Handling
1. **SIGINT Handling**: Your shell should catch the `SIGINT` signal (Ctrl+C). It should not terminate the shell but should forward the `SIGINT` signal to the currently running foreground process, if there is one.
2. **SIGCHLD Handling**: When a child process terminates, `SIGCHLD` will be sent to your shell. Your shell should forward `SIGCHLD` to the currently running process using `kill()`.
3. **Reaping Child Processes**: Use `waitpid()` in a loop to reap any child processes whenever `SIGCHLD` is received.

### Makefile

Provide a makefile that builds and runs your shell.  Provide test cases by running your shell with standard input redirected from a file of sample commands (a shell script!).
