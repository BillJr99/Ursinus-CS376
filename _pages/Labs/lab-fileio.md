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

## Getting Started

### Step 1: Define the Data Structure

First, we need to define a `struct` in C to represent each flight log entry. This struct will include fields for the date, aircraft type, departure and arrival locations, flight time, and any remarks. Since C does not have built-in string types, we'll use character arrays for storing these details. The length of each array should be chosen to accommodate the expected maximum size of each field.

```c
#define MAX_DATE_LEN 11
#define MAX_AIRCRAFT_TYPE_LEN 32
#define MAX_LOCATION_LEN 64
#define MAX_FLIGHT_TIME_LEN 6
#define MAX_REMARKS_LEN 256

typedef struct {
    char date[MAX_DATE_LEN];
    char aircraftType[MAX_AIRCRAFT_TYPE_LEN];
    char departure[MAX_LOCATION_LEN];
    char arrival[MAX_LOCATION_LEN];
    char flightTime[MAX_FLIGHT_TIME_LEN];
    char remarks[MAX_REMARKS_LEN];
} FlightLog;
```

### Step 2: Implement Add, Search, and Print Functions

### Adding New Flight Logs

To add a new flight log, we'll need a function that accepts user input for each field, creates a `FlightLog` `struct`, and appends it to a file. It's crucial to perform validation on user inputs to ensure data integrity.

```c
void addFlightLog(const char* filename) {
    FlightLog log;

    printf("Enter date (DD/MM/YYYY): ");
    scanf("%10s", log.date);
    printf("Enter aircraft type: ");
    scanf("%31s", log.aircraftType);
    printf("Enter departure location: ");
    scanf("%63s", log.departure);
    printf("Enter arrival location: ");
    scanf("%63s", log.arrival);
    printf("Enter flight time (HH:MM): ");
    scanf("%5s", log.flightTime);
    printf("Enter remarks: ");
    scanf(" %255[^\n]", log.remarks); // The space before % is to consume any newline left in the input buffer

    FILE *file = fopen(filename, "ab"); // Open the file in append binary mode
    if (file != NULL) {
        fwrite(&log, sizeof(FlightLog), 1, file);
        fclose(file);
    } else {
        printf("Failed to open the file.\n");
    }
}
```

### Searching Flight Logs by Date

To search for entries by date, iterate over each `FlightLog` entry in the file, comparing the input date with the date in the log.

```c
void searchFlightLogsByDate(const char* filename, const char* targetDate) {
    FILE *file = fopen(filename, "rb");
    FlightLog log;

    if (file == NULL) {
        printf("Failed to open the file.\n");
        return;
    }

    while(fread(&log, sizeof(FlightLog), 1, file)) {
        if (strcmp(log.date, targetDate) == 0) {
            // Print the matching log details
            printf("Match found: %s, %s, %s -> %s, %s, %s\n", log.date, log.aircraftType, log.departure, log.arrival, log.flightTime, log.remarks);
        }
    }

    fclose(file);
}
```

### Printing a Particular Record

To print a specific record by its number, calculate the byte offset in the file and use fseek to navigate to the correct position before reading the log entry.

```c
void printFlightLog(const char* filename, int recordNumber) {
    FILE *file = fopen(filename, "rb");
    FlightLog log;

    if (file == NULL) {
        printf("Failed to open the file.\n");
        return;
    }

    fseek(file, (recordNumber - 1) * sizeof(FlightLog), SEEK_SET);
    if (fread(&log, sizeof(FlightLog), 1, file)) {
        // Print the log details
        printf("Record %d: %s, %s, %s -> %s, %s, %s\n", recordNumber, log.date, log.aircraftType, log.departure, log.arrival, log.flightTime, log.remarks);
    } else {
        printf("Failed to read the record.\n");
    }

    fclose(file);
}
```

## Making a Deep Copy

When you write the contents of a struct, be careful if that struct contains `char*` data or other pointers.  `write` will simply write the address, and not the data!  This is called a **shallow copy**.  To perform a **deep copy**, you should `write` the actual fields of the struct one-by-one, and `read` them in the same way.  So, you'd have one `read` or `write` for each field of the `struct`.

In the provided code snippets, a deep copy issue is inherently avoided by using fixed-size character arrays instead of pointer-based strings (`char*`). This design choice ensures that when instances of `FlightLog` are written to or read from a file, the actual content of these arrays is directly handled, thus preserving the integrity of the data without the risks associated with shallow copies.

## Makefile

Be sure to include a Makefile with your submission that builds and tests your program.  If you prefer, you can include a shell script of test cases that you execute via the Makefile.

