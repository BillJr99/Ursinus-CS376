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

4. Modify this program to take in a linked list of `struct`s that you create (with an `int` data element, and a `struct ListNode*`), and sort the linked list. Note that you should swap the actual nodes and not just the values within those nodes.

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
