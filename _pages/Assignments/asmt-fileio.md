---
layout: assignment
permalink: Assignments/FileIO
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

In this assignment, you will open a text file, get its size, `malloc` a buffer, and read the contents of the file into that buffer.  

To read a string from the console, you can use the line `fgets(buf, 1024, stdin);` from `stdio.h`, assuming that `buf` is `malloc`'ed to at least `1025` bytes (I am leaving room for a null terminator in case you wish to print the buffer!).  Using this, prompt the user to enter a word to search the text file for.  

Loop over each line of the file contents by tokenizing it by the newline `\n` character.  For each line, tokenize that line by spaces.  Then, loop over this list of tokens and check if the token matches that line.  If it does, print the entire line to the screen.

Lastly, create a new file using `O_CREAT` and write these matching lines to the new text file (so that lines that do not match are skipped).  Be sure to open the file with `0666` permissions and with `O_RDWR` so that you can both read and write to the file.  

The filename should be given via `argv` command line parameters.  The filename to read will be in `argv[1]` and the filename to write will be in `argv[2]` (recall that the program name is `argv[0]`).  Be careful to check `argc` before accessing these values.  If `argc` is not at least 3, print an error message to the screen and `exit` or `return` a nonzero token value to indicate that an error has occurred.

`close` the file and `free` any memory when done.

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

## Makefile

Be sure to include a Makefile with your submission that builds and tests your program.  If you prefer, you can include a shell script of test cases that you execute via the Makefile.
