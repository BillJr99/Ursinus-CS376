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
    - rlink: "https://makefiletutorial.com/"
      rtitle: "Makefile Tutorial"
    - rlink: "https://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/"
      rtitle: "A Simple Makefile Tutorial"

tags:
  - minidb
  - files

---

In this assignment, you will write a mini-database program in the C language that uses basic file I/O.  You may choose any file format you like, including flat file, XML, JSON, comma separated values, binary files, or a format of your own choosing, but you must support the following:

* Some SQL-like operations, namely:
  * `SELECT * FROM TableName WHERE Field1="value"`
  * `DELETE FROM TableName WHERE Field1="value"`
  * `UPDATE TableName SET Field1="new value" WHERE Field2="value"`
  * `INSERT INTO TableName (Field1="value", Field2="value", ...)`
  
* Creation and deletion of multiple tables per database:
  * `CREATE TABLE TableName FIELDS [Field1, Field2, ...]`
  * `DROP TABLE TableName`
  
In addition, your program must be able to select a database, creating one if it does not exist, via a command-line parameter.

You must use only the standard library file operations (no CSV or json libraries, etc.).  You may use the CSAPP file I/O libraries.


## Makefile

Be sure to include a Makefile with your submission that builds and tests your program.  If you prefer, you can include a shell script of test cases that you execute via the Makefile.

## Getting Started

This section will guide you through the development of a mini-database system in C, first by directly manipulating comma-separated value (CSV) files, and then by using data structures to represent and manipulate records more efficiently. You may choose either approach (or something different!)

### Part 1: Using CSV Files for Data Storage and Manipulation

**Step 1: Understanding the CSV Format**

- CSV files are simple text files where each line represents a record and each field within a record is separated by a comma.
- The first line often serves as a header, listing field names.

**Step 2: Opening and Reading CSV Files**

- Use file I/O operations to open and read CSV files line by line, utilizing the standard I/O library functions.
- Be sure to handle file read errors and end-of-file (EOF) conditions gracefully by checking the return value of your system calls.

**Step 3: Parsing CSV Lines**

- Tokenize your file first by newline to get each line of the file.
- Tokenize each line using commas as delimiters to separate fields within a record.

**Step 4: Identifying the Field Index**

- Before manipulating data, you must find the index of the field (column) you wish to operate on, based on the header line.
- Loop through the line tokens (the words) until the word matches the field you're looking for.  Keep track of a counter as you iterate through the loop.
- That counter represents the field number for that column!  If you tokenize each subsequent line of the file, you can iterate to that index number and retrieve the value.
- Make this a helper function that returns the index of a field in the file given its column name in the header line.

**Step 5: Implementing SQL-like Operations**

- **SELECT Operation**: To perform a SELECT operation on a CSV file, you would read the file line by line, parse each line to tokenize the fields, and compare the relevant field's value against the criteria specified in the SELECT query. If a match is found, you would output or store the corresponding line.
- **INSERT Operation**: To INSERT a new record into a CSV file, you would append a new line to the end of the file. This involves converting the record's fields into a comma-separated string and writing this string, followed by a newline character, to the file.
- **UPDATE Operation**: Updating records in a CSV file requires reading the entire file, modifying the lines that meet the update criteria, and writing the updated content back to the file. This often means storing the modified file content in memory before rewriting it to the file.
- **DELETE Operation**: Similar to UPDATE, the DELETE operation involves reading the entire file, identifying the records that match the delete criteria, removing these records from the in-memory representation of the file, and then writing the modified content back to the file.

**Step 6: Closing Files and Memory Management**

- Close your file and `free` any `malloc`s when done.

### Part 2: Using Data Structures for Efficient Data Representation and Manipulation

**Step 1: Defining Data Structures**

- Begin by defining a `struct` in C to represent a record, with fields corresponding to columns you want to store.
- Consider dynamic memory allocation for storing an array of these structs, to accommodate a variable number of records.

**Step 2: Loading Data into Structures**

- For each record you intend to read from the file, allocate memory dynamically using `malloc()`. This allocation should be for a single instance of your struct.
- Keep a linked list of these structures, which represent your rows.

**Step 3: Implementing SQL-like Operations with Structures**

- **SELECT Operation**: To perform a SELECT operation on a linked list of record structs, iterate through the list, comparing the specified field in each struct against the selection criteria. For each struct that matches the criteria, you could perform an action such as displaying the record or adding it to a results list.
- **INSERT Operation**: To INSERT a new record into a linked list, create a new struct instance with the provided record data, allocate memory for it, and appropriately adjust the pointers in the list to include the new node. This might involve appending the new node at the end of the list or inserting it in a specific position based on some criteria.
- **UPDATE Operation**: For an UPDATE operation, traverse the linked list to find nodes (records) that match the update criteria. For each matching node, update the relevant fields with the new values. This operation requires no restructuring of the list itself, only modification of the selected nodes' data.
- **DELETE Operation**: To DELETE records from a linked list, iterate through the list, identifying nodes that match the delete criteria. For each node that matches, remove it from the list by adjusting the pointers of the adjacent nodes and then free the memory allocated for the removed node to avoid memory leaks.

**Step 4: Writing Data Back to the file**

- Save each data structure in your array back to the file.  You might find it helpful to delete the entire file and re-write everything, but you could also seek to the position of the record you're editing, and do a write of just that structure (which automatically overwrites the structure that was there before).

**Step 5: Closing Files and Memory Management**

- Close your file and `free` any `malloc`s when done.

## Helpful Utility Functions

### Tokenizing a String

You can use the `strtok` function to tokenize a string.  Here is a function that dynamically allocates (and re-allocates) an array of tokens given an input string, a string of possible delimiters, and an integer representing the number of tokens found (passed as a pointer so that updates are seen in the `main` function).

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char** tokenize(char* input, const char* delimiter, int* tokenCount) {
    char** tokens = NULL;
    int capacity = 10; // Initial capacity for the array of tokens
    *tokenCount = 0; // Initialize the token count

    // Allocate initial memory for tokens
    tokens = (char**)malloc(capacity * sizeof(char*));
    if (tokens == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return NULL;
    }

    // Duplicate the input string to avoid modifying the original string
    char* inputDup = strdup(input);
    if (inputDup == NULL) {
        free(tokens);
        fprintf(stderr, "Memory allocation failed\n");
        return NULL;
    }

    // Tokenize the input string
    char* token = strtok(inputDup, delimiter);
    while (token != NULL) {
        if (*tokenCount >= capacity) {
            // Increase capacity
            capacity *= 2;
            char** temp = (char**)realloc(tokens, capacity * sizeof(char*));
            if (temp == NULL) {
                // Cleanup and error handling
                for (int i = 0; i < *tokenCount; ++i) {
                    free(tokens[i]);
                }
                free(tokens);
                free(inputDup);
                fprintf(stderr, "Memory allocation failed\n");
                return NULL;
            }
            tokens = temp;
        }

        // Duplicate token and add to the array
        tokens[*tokenCount] = strdup(token);
        if (tokens[*tokenCount] == NULL) {
            // Cleanup and error handling
            for (int i = 0; i < *tokenCount; ++i) {
                free(tokens[i]);
            }
            free(tokens);
            free(inputDup);
            fprintf(stderr, "Memory allocation failed\n");
            return NULL;
        }

        (*tokenCount)++;
        token = strtok(NULL, delimiter);
    }

    free(inputDup); // Cleanup the duplicated string
    return tokens;
}

int main() {
    char input[] = "This is a test string, with several delimiters.";
    const char* delimiter = " ,."; // Tokenize based on space, comma, and period
    int tokenCount = 0;

    char** tokens = tokenize(input, delimiter, &tokenCount);
    if (tokens != NULL) {
        for (int i = 0; i < tokenCount; i++) {
            printf("%d: %s\n", i + 1, tokens[i]);
            free(tokens[i]); // Free each token
        }
        free(tokens); // Free the array of tokens
    }

    return 0;
}
```

### Comparing Two Strings

You can compare two strings using the `strcmp` function, which returns `0` when the strings are equal.  Here is an example:

```c
#include <stdio.h>
#include <string.h>

int main() {
    // Define pairs of strings to compare
    char* pairs[][2] = {
        {"apple", "apple"}, // Equal strings
        {"apple", "banana"}, // Lexicographically less
        {"banana", "apple"}, // Lexicographically greater
        {"apple", "Apple"}, // Case sensitivity, 'a' > 'A' in ASCII
        {"", ""}, // Both strings are empty
        {"apple", ""}, // Non-empty string compared with an empty string
        {"apple", "apple"} // The strings are equal
    };

    // Calculate the number of pairs
    int numPairs = sizeof(pairs) / sizeof(pairs[0]);

    for (int i = 0; i < numPairs; i++) {
        // Compare the strings
        int result = strcmp(pairs[i][0], pairs[i][1]);

        // Print the result of the comparison
        printf("Comparing \"%s\" and \"%s\": ", pairs[i][0], pairs[i][1]);
        if (result < 0) {
            printf("The first string is less than the second string.\n");
        } else if (result > 0) {
            printf("The first string is greater than the second string.\n");
        } else {
            printf("The strings are equal; result == 0.\n");
        }
    }

    return 0;
}
```

#### Optional: Converting a String to Lowercase

`strcmp` is a case-sensitive comparison, which is fine for this assignment.  If you like, you can convert your `char*` variables to lowercase so that these comparisons are effectively case-insensitive.  You can convert an individual `char` to lowercase using the `tolower(char)` function, which returns a new `char` and can be accessed by including `ctype.h`.  Using a loop, you could convert an entire string to lowercase.  Here is an example:

```c
#include <stdio.h>
#include <ctype.h>

void toLowerCase(char* s) {
    if (s == NULL) return; // Handle null pointer
    for (int i = 0; s[i] != '\0'; i++) {
        s[i] = tolower((unsigned char)s[i]);
    }
}

int main() {
    char str[] = "Hello, World!";
    printf("Original: %s\n", str);
    toLowerCase(str);
    printf("Lowercase: %s\n", str);
    return 0;
}
```
