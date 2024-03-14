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

### Fundamental Elements

#### Basic Shell Functionality
1. **Prompt**: The shell should display a prompt and wait for user input.
2. **Command Execution**: Your shell should allow the user to run commands with arguments. For example, `ls -l /home`.
3. **Input/Output Redirection**: Implement redirection of standard input and standard output. For instance, `cat < input.txt > output.txt`.  You do not need to support pipes.

#### Signal Handling
1. **SIGINT Handling**: Your shell should catch the `SIGINT` signal (Ctrl+C). It should not terminate the shell but should forward the `SIGINT` signal to the currently running foreground process, if there is one.
2. **SIGCHLD Handling**: When a child process terminates, `SIGCHLD` will be sent to your shell. Your shell should forward `SIGCHLD` to the currently running process using `kill()`.
3. **Reaping Child Processes**: Use `waitpid()` in a loop to reap any child processes whenever `SIGCHLD` is received.

#### Makefile

Provide a makefile that builds and runs your shell.  Provide test cases by running your shell with standard input redirected from a file of sample commands (a shell script!).

### What to Do

#### Step 1: The Loop and The Prompt

The main component of a shell is its loop, where it continuously displays a prompt, waits for user input, and executes the specified command. Here is a basic structure:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_LINE 1024 // Maximum length of a command
#define MAX_ARGS 16 // Maximum number of command line arguments

int main() {
    char cmd[MAX_LINE];
    char **args; // Command arguments
    int should_run = 1; // Flag to determine when to exit the program

    while (should_run) {
        printf("osh> "); // Display a prompt
        fflush(stdout);

        if (fgets(cmd, MAX_LINE, stdin) == NULL) {
            perror("fgets failed");
            exit(EXIT_FAILURE);
        }

        // Parse the command and its arguments from cmd
        // (Parsing logic / tokenizing goes here)
        // Hint: call tokenize on cmd so that the args array contains the individual words of the cmd line

        if (strcmp(args[0], "exit") == 0) { 
            // If the user typed in a built-in exit command, quit the shell
            should_run = 0; // Exit the loop
        } else {
            // Execute the command (Steps 2-4)
        }
    }

    return 0;
}
```

#### Step 2: Forking and Executing Commands

To execute a command, the shell should create a child process using `fork()`. This child process will then call `execvp()` to execute the command. The parent process waits for the child's execution to complete using `wait()`.

```c
#include <sys/wait.h>
#include <unistd.h>

// Inside the loop, after parsing the command:
if (should_run) {
    pid_t pid = fork();

    if (pid < 0) {
        perror("Fork failed");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // Child process
        if (execvp(args[0], args) < 0) {
            perror("Execvp failed");
            exit(EXIT_FAILURE);
        }
    } else {
        // Parent process
        int status;
        waitpid(pid, &status, 0); // wait on the child we just created
    }
}
```

#### Step 3: Handling Background Processes and SIGCHLD

To allow executing commands in the background, we modify the shell to handle `SIGCHLD` signals. This involves setting up a signal handler that reaps child processes using `waitpid()`.

```c
#include <signal.h>

void sigchld_handler(int signum) {
    // Wait for all children without blocking
    while (waitpid((pid_t)(-1), 0, WNOHANG) > 0) {}
}

int main() {
    // Set up the signal handler
    signal(SIGCHLD, sigchld_handler);
    
    // Main loop and other logic...

    return 0;
}
```

To actually run a process in the background, we will modify the `main` loop so that the parent does not call `wait` on the child immediately after `fork`ing it (since we will do this with `waitpid` when the child signals that it has terminated with the logic above).  Instead the parent can return to printing its prompt and proceeding as normal.

To do this, check if the last token in your input is the `&` character.  If it is, remove it from the list of tokens, and set a flag variable that indicates that the parent should not `waitpid` on the child like we did above.

Finally, in your main loop, when you start a process in the foreground, set `foreground_pid` to the pid of that child.  Be sure to set it back to `0` when that child terminates (which you'll know by checking the pid that is returned from your call to `wait`).  If you start a process in the background, do not set `foreground_pid`.  We will use this later to keep track of which process is the foreground process (because it should receive signals sent to the shell).

#### Step 4: SIGINT Handling

Your shell should catch `SIGINT` signals (`Ctrl+C`) and forward them to the foreground process, if any. This prevents the shell itself from being terminated by `SIGINT`.  Instead, it should forward the signal to the foreground process (if there is one) using the `kill` function.

```c
void sigint_handler(int signum) {
    // If there is a foreground process, send it SIGINT
    if (foreground_pid > 0) {
        kill(foreground_pid, SIGINT);
    }
}

int main() {
    signal(sigint, sigint_handler);

    // Main loop and other logic...

    return 0;
}
```

#### Step 5: Input/Output Redirection

To implement I/O redirection, you'll need to recognize the redirection symbols (`<`, `>`) in the command input, modify the file descriptors accordingly using `dup2()`, and then execute the command.

```c
// Inside the child process, before execvp():
int in, out;
if (need_input_redirection) {
    in = open(input_file, O_RDONLY);
    if (in < 0) {
        perror("Opening input file failed");
        exit(EXIT_FAILURE);
    }
    dup2(in, STDIN_FILENO); // replace stdin with the file
    close(in);
}

if (need_output_redirection) {
    out = open(output_file, O_WRONLY | O_CREAT | O_TRUNC, 0600);
    if (out < 0) {
        perror("Opening output file failed");
        exit(EXIT_FAILURE);
    }
    dup2(out, STDOUT_FILENO); // replace stdout with the file
    close(out);
}
```

Remember to check for the need for redirection by checking your tokens for the `<` or `>` character, and parse the input and output file names from the command input. You will remove the `<` or `>` token and the next token from the command line argument list if you encounter them.  This redirection logic should be placed in the child process code, right before the call to `execvp()`.
