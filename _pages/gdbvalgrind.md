---
layout: default
permalink: /GDBValgrind
title: "CS376: Operating Systems - Debugging and Memory Management in C with GDB and Valgrind"
excerpt: "CS376: Operating Systems - Debugging and Memory Management in C with GDB and Valgrind"

---

# Helpful Debugging Tools: gdb and Valgrind

## Using gdb to Step Through your Code

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

### Step 1: Compile the program with debugging symbols.
```
gcc -g -o sum sum.c
```

### Step 2: Launch GDB.
```
gdb ./sum
```

### Step 3: Set breakpoints.
Set a breakpoint at the beginning of the `main` function.
```
break main
```

### Step 4: Run the program.
```
run
```

### Step 5: Start stepping through the program.
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

### Step 6: Examine variables and print their values.
While stepping through the code, you can use the `print` command (abbreviated as `p`) to print the value of a variable.
```
print a
print b
```

### Step 7: Identify and fix the bug.
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

### Step 8: Set a watchpoint.
You can set a watchpoint on a variable to break whenever its value changes. In this example, let's set a watchpoint on the variable `result`.
```
watch result
```

### Step 9: Continue stepping through the program.
Continue stepping through the program until the watchpoint is triggered. Hit `next` twice to skip over the `printf` statement.
```
next
next
```

The watchpoint will be triggered when the `result` variable changes its value.

### Step 10: Print the new value of `result`.
```
print result
```

Here, you will see the new value of `result` after the correction.

### Step 11: Continue debugging.
After printing the new value of `result`, you can tell GDB to continue executing the program until it finishes or hits another breakpoint using the `continue` command (abbreviated as `c`).
```
continue
```

## Using gdb to Print a Stack Trace from a Segmentation Fault

To enable core dumping and analyze the core file using GDB, follow these steps:

### Step 1: Set unlimited core file size.
On Unix-like systems, you can use the `ulimit` command to set the maximum size of the core file. To allow an unlimited core file size, run the following command:
```
ulimit -c unlimited
```

### Step 2: Compile your program with debugging symbols.
Make sure to compile your program with the `-g` flag to include debugging information. For example:
```
gcc -g -o program program.c
```

### Step 3: Run the program and generate a core dump.
Execute your program as you normally would. If it crashes, it will generate a core dump file. The core dump file contains the program's memory and register state at the time of the crash.

### Step 4: Load the core file into GDB.
To analyze the core dump with GDB, use the following command:
```
gdb program core
```

Note that if you're using the Windows Subsystem for Linux (WSL), the core file is typically stored at `/mnt/wslg/dumps` on the Windows file system. On most systems, it will appear in the current directory.

### Step 5: Obtain a backtrace.
Once GDB is loaded with the core file, you can obtain a backtrace of the program's execution at the time of the crash using the `bt` command:
```
bt
```

The `bt` command will show the function call stack, which can help you identify the location and sequence of function calls leading up to the crash.

## Checking for Memory Leaks and Memory Errors with Valgrind

To check for memory leaks and other memory errors using Valgrind, follow these steps:

### Step 1: Compile your program with debugging symbols.
Make sure to compile your program with the `-g` flag when using GCC. This includes debugging information in the executable, which is necessary for Valgrind to provide full line information in the report. For example:
```
gcc -g -o program program.c
```

### Step 2: Run your program with Valgrind.
Execute your program with Valgrind using the following command:
```
valgrind --leak-check=full ./program
```

Valgrind's `--leak-check=full` flag enables detailed memory leak reporting, highlighting any memory blocks that were not freed. This can help identify memory leaks caused by failing to call `free` or other memory management errors.  Valgrind will run your program and monitor memory operations, checking for memory leaks, buffer overflows, and other memory errors.  After running your program, Valgrind will generate a report detailing any memory errors or leaks it found. It will provide information about the source code location where the error occurred, including the file name and line number.
