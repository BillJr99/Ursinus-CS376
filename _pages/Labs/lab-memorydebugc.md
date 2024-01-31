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
    - rlink: ../GDBValgrind
      rtitle: "Helpful Debugging Tools: GDB and Valgrind"
    - rlink: https://valgrind.org/docs/manual/quick-start.html
      rtitle: Valgrind
    - rlink: https://www.geeksforgeeks.org/gdb-step-by-step-introduction/
      rtitle: GDB Step by Step Introduction

tags:
  - c
  - pointers

---

## Part 1: C Programming and Pointer Arithmetic

1. Define an `int*` pointer variable, and create an array of 10 integers using `malloc()`.  Then, assign values to that array, print their values, and `free()` the integers.  Verify that your program does not leak memory by running it against `valgrind` (and include the report you get from `valgrind` in your submission).

2. Using `malloc()`, create a `char**` pointer that contains 10 `char*`'s, then in a loop, initialize each of the 10 `char*`'s to a `char` array of size 15, and initialize each to a word of your choice (don't forget the null terminator `\0`) -- and print them to screen.  Again, free all your `malloc`'s, and verify the result with `valgrind`.

3. Write a function `sort()` that takes in an `int* a` and `int size`, and sorts the array using pointer arithmetic.  That is, you should not use array notation `a[n]` to refer to elements in your array, but rather direct pointer addressing `*(a+n)`

4. Write another function to take in a linked list of `struct`s that you create (with an `int` data element, and a `struct ListNode*`), and sort the linked list. Note that you should swap the actual nodes and not just the values within those nodes.  Consider the following scenario using a doubly linked list (a linked list with both `prev` and `next` pointers so that you can point to both your previous and next neighbors).  If you turned this logic into a subroutine called, say, `swap`, then you could pass nodes `B` and `D` to this function and update the entire list.  Keep in mind, however, that in the general case, some of the neighbors of these nodes may be `NULL` (for example, if the node you are swapping is at the beginning or end of the list), and you'll want to check for and test these boundary cases.  Draw yourself a diagram of this scenario, showing each step, and draw diagrams for the boundary cases to see how they differ.

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

I recommend creating a `struct` to represent your `ArrayList`, which contains a pointer to the array, the current size of the array (for example, large enough to hold 10 `int`s), and the number of elements currently stored in the array (initially `0`).

Write a program that, using `malloc()` and `realloc()`, creates an empty array of initial size `n=10`.  Write `add()`, `remove()` and `get()` functions for your structure which manipulate the array accordingly.  When adding beyond the end of the array, reallocate space such that the array contains one more element.  

### Reallocating the Array Size Dynamically

Note that the `add` function might call `realloc`, and if so, you'll update the parameter for your maximum array size.  For now, increase the maximum size by `1` every time you need to `realloc`.

#### Being Careful about Modifying Function Parameters

C uses pass-by-value semantics, so modifications to local parameters are made to local copies of those parameter values; they do not update the variable in the calling function.  So, if you are using a `struct` to represent your `ArrayList`, you can simply pass a pointer to that structure, and you can freely update the fields inside that `struct`.  You can, for example, set `myArrayList->array = realloc...` assuming `myArrayList` is a `struct ArrayList *`, and those changes will persist when your function returns.

However, if you pass the array directly, you'll need pass the array as an `int**` -- a pointer to the variable that contains the pointer to your array.  This way, you can modify the parameter as if it was passed by reference.  Although you can pass an `int *` array parameter to a function and make statements like `a[i] = 0;`, this is because this statement is equivalent to `*(a+i) = 0;`, which does indeed dereference the pointer to the address!  A good rule of thumb is that if you ever have a statement like `x = something;` where `x` is a local parameter, you'll want to dereference it so that the modification persists beyond the function call.  That means passing a pointer to the parameter, even if the parameter itself is already a pointer.  So, if `x` is an `int`, pass an `int *`, and if `x` is an `int *`, pass an `int**`.  Then, update your statement to `*x = something;`.

### Experimenting with the Program's Performance

Time your program (using the command `time ./a.out ...`) for adding `100000` elements (or more).  

Then, modify the program such that it increases in size by a factor of `2` times the previous size.  Time it again.  What do you observe?

### Extra Credit (15%): Adding Items to and Removing Items from the Middle of an Array

It is fine to simply add and remove from the end of the array for this assignment.  If you are adding or removing to the middle of an array, you'll need to shift the remaining elements right or left to make room for the newly added element, or to remove the outgoing one.  The `memcpy` function can help you to shift the array elements around.  Consider the function prototype:

```c
#include <stdlib.h>
void *memcpy(void *dest, const void *src, size_t n);
```

It is not guaranteed to be safe to copy from one region of memory into another overapping region, but we can `malloc` a temporary buffer of size `sizeof(int) * (size-n)` to hold the elements to be moved.  You could then copy from `arr+n+1` to this buffer, and copy `sizeof(int) * (size-n)` bytes to that buffer (the rightmost `size-n` elements of the array.  Then, `memcpy` to the original array at address `arr+n` as the `dest`, from this temporary buffer as the source (and the same number of bytes as before), to copy the rightmost `size-n` integers into the original array, effectively shifting those elements to the left by one element.  Don't forget to update the number of elements in the array to `num_elements-1` to reflect that an item has been removed.  

Similarly, to add an item to the middle, you could shift those `size-n` elements to the right by one position by following the same general steps above (again, updating the number of elements, and possibly resizing the entire array as needed).
