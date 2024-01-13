---
layout: assignment
permalink: Assignments/FileIO
title: "CS376: Operating Systems - File I/O"
excerpt: "CS376: Operating Systems - File I/O"

info:
  coursenum: CS376
  points: 100
  goals:
    - Synthesize a functional mini-database program in C, demonstrating creativity in choosing and implementing a file format (e.g., flat file, XML, JSON, CSV, binary, or custom format).
    - Analyze standard library file operations (or CSAPP file I/O libraries) and apply these concepts effectively to handle data storage and retrieval.
    - Develop a mechanism that allows the selection of a database through a command-line parameter, incorporating the functionality to create a new database if it does not exist.
    - Create and modify files using the C standard I/O interface

  rubric:
    - weight: 30
      description: Implementation of SQL-like Operations
      preemerging: No implementation of SQL-like operations.
      beginning: Basic implementation of one or two SQL-like operations with significant issues. 
      progressing: Partial implementation of most SQL-like operations with minor issues. 
      proficient: Full implementation of all specified SQL-like operations functioning correctly. 
    - weight: 20
      description: Design and Construction of File I/O System
      preemerging: Ineffective or non-existent file I/O system. 
      beginning: Basic file I/O functionality with significant limitations or errors. 
      progressing: Functional file I/O system with some limitations or minor errors. 
      proficient: Robust and efficient file I/O system with no significant issues. 
    - weight: 20
      description: Database Table Management (Create and Drop Table)
      preemerging: No functionality for creating or dropping tables. 
      beginning: Basic functionality for table creation or deletion with major issues. 
      progressing: Good functionality for managing tables, minor issues in creation or deletion processes. 
      proficient: Fully functional and efficient table management system, including creation and deletion. 
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

tags:
  - minidb
  - files

---

In this assignment, you will write a mini-database program in the C language that uses basic file I/O.  You may choose any file format you like, including flat file, XML, JSON, comma separated values, binary files, or a format of your own choosing, but you must support the following:

* Some SQL-like operations, namely:
  * `SELECT * FROM TableName WHERE Field1="value"`
  * `DELETE FROM TableName WHERE Field1="value"`
  * `UPDATE TableName SET Field1="new value" WHERE Field2="value"`
  * `SELECT * FROM TableName JOIN TableName2 ON TableName.Field1 = TableName2.Field2`
  * `INSERT INTO TableName (Field1="value", Field2="value", ...)`
  
* Creation and deletion of multiple tables per database:
  * `CREATE TABLE TableName FIELDS [Field1, Field2, ...]`
  * `DROP TABLE TableName`
  
In addition, your program must be able to select a database, creating one if it does not exist, via a command-line parameter.

You must use only the standard library file operations (no CSV or json libraries, etc.).  You may use the CSAPP file I/O libraries.

## Makefile

Be sure to include a Makefile with your submission that builds and tests your program.  If you prefer, you can include a shell script of test cases that you execute via the Makefile.
