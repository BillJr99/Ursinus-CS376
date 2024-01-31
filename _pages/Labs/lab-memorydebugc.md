---
layout: assignment
permalink: Labs/MemoryDebugC
title: "CS376: Operating Systems - Debugging and Memory Management in C"
excerpt: "CS376: Operating Systems - Debugging and Memory Management in C"

info:
  coursenum: CS376
  points: 100
  goals:
    - Apply dynamic memory allocation for primitive data types using malloc()
    - Manage memory for arrays of strings
    - Apply pointer arithmetic to implement and reinforce sorting algorithms
    - Manipulate linked lists using their underlying memory structures
    - Demonstrate the underlying memory implementation of dynamic array structures
    - Demonstrate that arrays cannot grow on their own because they are fixed in memory

  rubric:
    - weight: 50
      description: Program Correctness
      preemerging: Program does not compile or runs with critical errors that prevent functionality testing.
      beginning: Program compiles but has major functional errors or missing key components.
      progressing: Program runs with minor errors. Basic functionality is implemented, but there are issues in output or behavior.
      proficient: Program runs correctly with no errors. All specified functionalities are correctly implemented and produce accurate results.
    - weight: 20
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
    - weight: 20
      description: Implementation of Arraylist Structures
      preemerging: Incomplete or incorrect implementation of both versions of the arraylist structure.
      beginning: Basic implementation of one version of the arraylist structure, but with significant errors or omissions.
      progressing: Correct implementation of one version of the arraylist structure, or partial implementation of both versions.
      proficient: Correct and efficient implementation of both versions of the arraylist structure.

  readings:
    - rlink: http://www.tutorialspoint.com/cprogramming/index.htm
      rtitle: C Programming Tutorial

tags:
  - c
  - pointers

---

## Part 1: C Programming and Pointer Arithmetic

1. Define an `int*` pointer variable, and create an array of 10 integers using `malloc()`.  Then, assign values to that array, print their values, and `free()` the integers.  Verify that your program does not leak memory by running it against `valgrind` (and include the report you get from `valgrind` in your submission).

2. Using `malloc()`, create a `char**` pointer that contains 10 `char*`'s, then in a loop, initialize each of the 10 `char*`'s to a `char` array of size 15, and initialize each to a word of your choice (don't forget the null terminator `\0`) -- and print them to screen.  Again, free all your `malloc`'s, and verify the result with `valgrind`.

3. Write a function `sort()` that takes in an `int* a` and `int size`, and sorts the array using pointer arithmetic.  That is, you should not use array notation `[n]` to refer to elements in your array, but rather direct pointer addressing `*(a+n)`

4. Write another function to take in a linked list of `struct`s that you create (with an `int` data element, and a `struct ListNode*`), and sort the linked list. Note that you should swap the actual nodes and not just the values within those nodes.  Consider the following scenario using a doubly linked list (a linked list with both `prev` and `next` pointers so that you can point to both your previous and next neighbors).  Keep in mind, however, that in the general case, some of these nodes may be NULL (for example, if the node you are swapping is at the beginning or end of the list), and you'll want to check for and test these boundary cases.  Draw yourself a diagram of this scenario, showing each step, and draw diagrams for the boundary cases to see how they differ.

    * The initial doubly linked List is: `A->B->C->D->E`, and we wish to swap nodes B and D.
        - A: next = B
        - B: prev = A, next = C
        - C: prev = B, next = D
        - D: prev = C, next = E
        - E: prev = D

    * Update the neighbors of B and D:
        - A's next should point to D, since D will follow A after the swap (where B used to be)
        - E's prev should point to B, since B will come before E after the swap (where D used to be)
        - C's prev should point to D, since D will come before C after the swap (where B used to be)
        - C's next should point to B, since B will come after C after the swap (where D used to be)
        
    * Update the pointers from B and D:
        - B's prev should become D's current prev, which is C
        - B's next should be D's current next, which is E
        - D's prev should beocme B's current prev, which is A
        - D's next should be B's current next, which is C
     
    * The final list structure is: `A->D->C->B->E`, with the following pointers:
        - A: next = D
        - D: prev = A, next = C
        - C: prev = D, next = B
        - B: prev = C, next = E
        - E: prev = B 

## Part 2: Implementing the `ArrayList` Structure

Write a program that, using `malloc()` and `realloc()`, creates an array of initial size `n`.  Write `add()`, `remove()` and `get()` functions for your array.  When adding beyond the end of the array, reallocate space such that the array contains one more element.  

Note that the `add` function might call `realloc`, and if so, you'll update the parameter for your array.  C uses pass-by-value semantics, so modifications to local parameters are made to local copies of those parameter values; they do not update the variable in the calling function.  Therefore, you'll pass the array as an `int**` -- a pointer to the variable that contains the pointer to your array.  This way, you can modify the parameter as if it was passed by reference.  

Time your program (using the command `time ./a.out ...`) for adding `100000` elements (or more).  

Next, modify the program such that it increases in size by a factor of `2` times the previous size.  Time it again.  What do you observe?

### Extra Credit (15%): Adding Items to and Removing Items from the Middle of an Array

It is fine to simply add and remove from the end of the array for this assignment.  If you are adding or removing to the middle of an array, you'll need to shift the remaining elements right or left to make room for the newly added element, or to remove the outgoing one.  The `memcpy` function can help you to shift the array elements around.  Consider the function prototype:

```c
#include <stdlib.h>
void *memcpy(void *dest, const void *src, size_t n);
```

It is not guaranteed to be safe to copy from one region of memory into another overapping region, but we can `malloc` a temporary buffer of size `sizeof(int) * (size-n)` to hold the elements to be moved.  You could then copy from `arr+n+1` to this buffer, and copy `sizeof(int) * (size-n)` bytes to that buffer (the rightmost `size-n` elements of the array.  Then, `memcpy` to the original array at address `arr+n` as the `dest`, from this temporary buffer as the source (and the same number of bytes as before), to copy the rightmost `size-n` integers into the original array, effectively shifting those elements to the left by one element.  Don't forget to update the number of elements in the array to `num_elements-1` to reflect that an item has been removed.  

Similarly, to add an item to the middle, you could shift those `size-n` elements to the right by one position by following the same general steps above (again, updating the number of elements, and possibly resizing the entire array as needed).

## Optional: Helpful Debugging Tools: gdb and Valgrind

### Using gdb to Step Through your Code

Consider the following corrected sample program that calculates the sum of two numbers (with a bug in the code):

```c
#include <stdio.h>

int add(int a, int b) {
    return a - b;
}

int main() {
    int a = 5;
    int b = 10;
    int result = add(a, b);
    printf("The sum of %d and %d is %d\n", a, b, result);
    return 0;
}
```

Let's walk through the step-by-step debugging process using GDB:

Step 1: Compile the program with debugging symbols.
```
gcc -g -o sum sum.c
```

Step 2: Launch GDB.
```
gdb ./sum
```

Step 3: Set breakpoints.
Set a breakpoint at the beginning of the `main` function.
```
break main
```

Step 4: Run the program.
```
run
```

Step 5: Start stepping through the program.
Once the program hits the breakpoint, you can start stepping through the code to identify the bug.

- **Step over (continue execution):**
If you want to step over a system call or a function call without diving into its implementation, use the `next` command (abbreviated as `n`).
```
next
```

- **Step into (dive into a function):**
If you want to step into a function and examine its implementation, use the `step` command (abbreviated as `s`).
```
step
```

Step 6: Examine variables and print their values.
While stepping through the code, you can use the `print` command (abbreviated as `p`) to print the value of a variable.
```
print a
print b
```

Step 7: Identify and fix the bug.
In this example, let's assume there's a bug in the `add` function, where instead of adding the numbers, it subtracts them. We will identify and fix this bug using GDB.

- Step into the `add` function:
```
step
```

- Print the values of `a` and `b` within the `add` function:
```
print a
print b
```

Here, you will see that the values of `a` and `b` are 5 and 10, respectively.

- Fix the bug by changing the `add` function implementation:
```c
int add(int a, int b) {
    return a + b; // Change the addition operator to subtraction
}
```

Step 8: Set a watchpoint.
You can set a watchpoint on a variable to break whenever its value changes. In this example, let's set a watchpoint on the variable `result`.
```
watch result
```

Step 9: Continue stepping through the program.
Continue stepping through the program until the watchpoint is triggered. Hit `next` twice to skip over the `printf` statement.
```
next
next
```

The watchpoint will be triggered when the `result` variable changes its value.

Step 10: Print the new value of `result`.
```
print result
```

Here, you will see the new value of `result` after the correction.

Step 11: Continue debugging.
After printing the new value of `result`, you can tell GDB to continue executing the program until it finishes or hits another breakpoint using the `continue` command (abbreviated as `c`).
```
continue
```

### Using gdb to Print a Stack Trace from a Segmentation Fault

To enable core dumping and analyze the core file using GDB, follow these steps:

Step 1: Set unlimited core file size.
On Unix-like systems, you can use the `ulimit` command to set the maximum size of the core file. To allow an unlimited core file size, run the following command:
```
ulimit -c unlimited
```

Step 2: Compile your program with debugging symbols.
Make sure to compile your program with the `-g` flag to include debugging information. For example:
```
gcc -g -o program program.c
```

Step 3: Run the program and generate a core dump.
Execute your program as you normally would. If it crashes, it will generate a core dump file. The core dump file contains the program's memory and register state at the time of the crash.

Step 4: Load the core file into GDB.
To analyze the core dump with GDB, use the following command:
```
gdb program core
```

Note that if you're using the Windows Subsystem for Linux (WSL), the core file is typically stored at `/mnt/wslg/dumps` on the Windows file system. On most systems, it will appear in the current directory.

Step 5: Obtain a backtrace.
Once GDB is loaded with the core file, you can obtain a backtrace of the program's execution at the time of the crash using the `bt` command:
```
bt
```

The `bt` command will show the function call stack, which can help you identify the location and sequence of function calls leading up to the crash.

### Checking for Memory Leaks and Memory Errors with Valgrind

To check for memory leaks and other memory errors using Valgrind, follow these steps:

Step 1: Compile your program with debugging symbols.
Make sure to compile your program with the `-g` flag when using GCC. This includes debugging information in the executable, which is necessary for Valgrind to provide full line information in the report. For example:
```
gcc -g -o program program.c
```

Step 2: Run your program with Valgrind.
Execute your program with Valgrind using the following command:
```
valgrind --leak-check=full ./program
```

Valgrind will run your program and monitor memory operations, checking for memory leaks, buffer overflows, and other memory errors.

Step 3: Analyze the Valgrind report.
After running your program, Valgrind will generate a report detailing any memory errors or leaks it found. It will provide information about the source code location where the error occurred, including the file name and line number.

Valgrind's `--leak-check=full` flag enables detailed memory leak reporting, highlighting any memory blocks that were not freed. This can help identify memory leaks caused by failing to call `free` or other memory management errors.
