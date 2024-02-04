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
  * `SELECT * FROM TableName JOIN TableName2 ON TableName.Field1 = TableName2.Field2`
  * `INSERT INTO TableName (Field1="value", Field2="value", ...)`
  
* Creation and deletion of multiple tables per database:
  * `CREATE TABLE TableName FIELDS [Field1, Field2, ...]`
  * `DROP TABLE TableName`
  
In addition, your program must be able to select a database, creating one if it does not exist, via a command-line parameter.

You must use only the standard library file operations (no CSV or json libraries, etc.).  You may use the CSAPP file I/O libraries.

## Tokenizing a String

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

## Comparing Two Strings

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

### Converting a String to Lowercase

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

## Makefile

Be sure to include a Makefile with your submission that builds and tests your program.  If you prefer, you can include a shell script of test cases that you execute via the Makefile.
