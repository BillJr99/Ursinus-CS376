---
layout: assignment
permalink: Labs/FileIO
title: "CS376: Operating Systems - File I/O"

info:
  coursenum: CS376
  points: 100
  goals:
    - Create and modify files using the C standard I/O interface

  rubric:
    - weight: 70
      description: Program Correctness
      preemerging: Code does not compile or execute, or produces critical errors.
      beginning: Code compiles and executes but contains significant logic errors, leading to incorrect results.
      progressing: Code compiles, executes, and produces correct results in most cases but may have minor issues.
      proficient: Code is correct, produces accurate results, and handles edge cases effectively. 
    - weight: 15
      description: Makefile and Compilation
      preemerging: No Makefile provided or compilation issues. 
      beginning: Basic Makefile provided but with errors or missing targets. 
      progressing: Functional Makefile, minor issues with targets or dependencies. 
      proficient: Well-structured Makefile, enabling seamless compilation without errors. 
    - weight: 15
      description: Documentation (README)
      preemerging: No README or severely lacking in information. 
      beginning: Basic README provided but lacking essential information or clarity. 
      progressing: Good README, covers most essential aspects with minor omissions or clarity issues. 
      proficient: Comprehensive, clear, and detailed README, providing complete instructions on program usage. 

  readings:
    - rlink: https://csapp.cs.cmu.edu/3e/ics3/code/include/csapp.h
      rtitle: "csapp.h"
    - rlink: https://www.cs.cmu.edu/~213/code/22-netprog1/csapp.c
      rtitle: "csapp.c"
    - rlink: https://condor.depaul.edu/glancast/374class/docs/csapp_compile_guide.html
      rtitle: "Compiling with the CSAPP Library"
    - rlink: "https://makefiletutorial.com/"
      rtitle: "Makefile Tutorial"
    - rlink: "https://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/"
      rtitle: "A Simple Makefile Tutorial"

tags:
  - files

---

The goal is to create a simple Flight Logbook System that allows users (pilots) to store and manage their flight records. This system should enable the addition, modification, viewing, and searching of flight logs. Each log entry will represent a flight, containing details such as date, aircraft type, departure and arrival locations, flight time, and any remarks.

Begin by creating a data structure that stores the following items:

* Date
* Aircraft Type
* Departure
* Arrival
* Flight Time
* Remarks

## What to Do

1. Allow users to add new flight logs by entering the details above.  Create a data structure containing these entries, and append that data structure to the file.
2. Allow the user to search for entries given a date (which you can compare as a string).  Read each data structure from the file and compare the data to the input, printing the record data if it matches.
3. Allow the user to print out a particular record number.  Seek to that record number (don't forget to multiply it by the size of each structure), and print the details at that entry.

## Deep Copy

When you write the contents of a struct, be careful if that struct contains `char*` data or other pointers.  `write` will simply write the address, and not the data!  This is called a **shallow copy**.  To perform a **deep copy**, you should `write` the actual fields of the struct one-by-one, and `read` them in the same way.  So, you'd have one `read` or `write` for each field of the `struct`.

## Makefile

Be sure to include a Makefile with your submission that builds and tests your program.  If you prefer, you can include a shell script of test cases that you execute via the Makefile.

