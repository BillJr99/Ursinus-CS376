---
layout: assignment
permalink: Labs/GdbValgrind
title: "CS376: Operating Systems - Using gdb and Valgrind"

info:
  coursenum: CS376
  points: 100
  goals:
    - To use classic debugging tools such as gdb and valgrind.

  rubric:
    - weight: 40
      description: Program Correctness
      preemerging: Program does not compile or runs with critical errors that prevent functionality testing.
      beginning: Program compiles but has major functional errors or missing key components.
      progressing: Program runs with minor errors. Basic functionality is implemented, but there are issues in output or behavior.
      proficient: Program runs correctly with no errors. All specified functionalities are correctly implemented and produce accurate results.
    - weight: 40
      description: Memory Management
      preemerging: Numerous memory leaks and errors. No evidence of effort to manage memory correctly.
      beginning: Some memory leaks are evident. Minimal effort to manage memory is observed.
      progressing: Minor memory leaks. Demonstrates an understanding of memory management, but with some oversights.
      proficient: No memory leaks. Efficient and correct use of memory management techniques. Valgrind report is clean with no issues reported.
    - weight: 10
      description: Valgrind Reports
      preemerging: No Valgrind report submitted or report shows critical memory management issues.
      beginning: Valgrind report submitted, but shows several memory management issues.
      progressing: Valgrind report indicates minor memory issues, demonstrating some proficiency in memory management.
      proficient: Clean Valgrind report with no memory errors or leaks, indicating thorough and effective memory management.
    - weight: 10
      description: Writeup and Submission
      preemerging: An incomplete submission is provided
      beginning: The program is submitted, but not according to the directions in one or more ways (for example, because it is lacking a readme writeup or missing answers to written questions)
      progressing: The program is submitted according to the directions with a minor omission or correction needed, including a readme writeup describing the solution and answering nearly all questions posed in the instructions
      proficient: The program is submitted according to the directions, including a readme writeup describing the solution and answering all questions posed in the instructions
      
  readings:  
    - rlink: http://www.tutorialspoint.com/cprogramming/index.htm
      rtitle: C Programming Tutorial
    - rlink: ../GDBValgrind
      rtitle: "Helpful Debugging Tools: GDB and Valgrind"
    - rlink: ../UbuntuDocker
      rtitle: "Launching an Ubuntu virtual machine with Docker"      
    - rlink: https://valgrind.org/docs/manual/quick-start.html
      rtitle: Valgrind
    - rlink: https://www.geeksforgeeks.org/gdb-step-by-step-introduction/
      rtitle: GDB Step by Step Introduction

tags:
  - c
  - pointers

---

The primary goal of this assignment is to familiarize students with using debugging tools such as gdb (GNU Debugger) and Valgrind to identify and fix common programming errors in C/C++ code. Students will be provided with intentionally buggy code samples that contain typical errors such as writing past the end of an array and dereferencing a null pointer. They will use gdb to step through the code and Valgrind to analyze memory usage and errors, ultimately correcting the bugs.

For each example, run the program, and run it through the step debugger and valgrind to determine the bugs.  Provide the commands you ran and the reports you got, describe the problem, and fix the code!

## Part 1

```c
#include <stdio.h>

int main() {
    char buffer[10];
    for (int i = 0; i <= 10; i++) {
        buffer[i] = 'a'; 
    }
    printf("Buffer: %s\n", buffer);
    return 0;
}
```

## Part 2

```c
#include <stdio.h>
#include <string.h>

int main() {
    char src[] = "hello";
    char dest[5]; 
    strcpy(dest, src); 
    printf("Dest: %s\n", dest);
    return 0;
}
```

## Part 3

```c
#include <stdio.h>

int main() {
    char *ptr = NULL; 
    printf("%c\n", *ptr); 
    return 0;
}
```