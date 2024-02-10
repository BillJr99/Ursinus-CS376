---
layout: assignment
permalink: Assignments/ShellScripting
title: "CS376: Operating Systems - Shell Scripting"

info:
  coursenum: CS376
  points: 100
  goals:
    - To use standard UNIX shell tools
    - To write a Bash shell script

  rubric:
    - weight: 60
      description: Algorithm Implementation
      preemerging: The algorithm fails on the test inputs due to major issues, or the program fails to compile and/or run
      beginning: The algorithm fails on the test inputs due to one or more minor issues
      progressing: The algorithm is implemented to solve the problem correctly according to given test inputs, but would fail if executed in a general case due to a minor issue or omission in the algorithm design or implementation
      proficient: A reasonable algorithm is implemented to solve the problem which correctly solves the problem according to the given test inputs, and would be reasonably expected to solve the problem in the general case
    - weight: 30
      description: Code Quality and Documentation
      preemerging: Code commenting and structure are absent, or code structure departs significantly from best practice, and/or the code departs significantly from the style guide
      beginning: Code commenting and structure is limited in ways that reduce the readability of the program, and/or there are minor departures from the style guide
      progressing: Code documentation is present that re-states the explicit code definitions, and/or code is written that mostly adheres to the style guide
      proficient: Code is documented at non-trivial points in a manner that enhances the readability of the program, and code is written according to the style guide
    - weight: 10
      description: Writeup and Submission
      preemerging: An incomplete submission is provided
      beginning: The program is submitted, but not according to the directions in one or more ways (for example, because it is lacking a readme writeup or missing answers to written questions)
      progressing: The program is submitted according to the directions with a minor omission or correction needed, including a readme writeup describing the solution and answering nearly all questions posed in the instructions
      proficient: The program is submitted according to the directions, including a readme writeup describing the solution and answering all questions posed in the instructions
  readings:
    - rlink: https://linuxconfig.org/bash-scripting-tutorial
      rtitle: "Bash Scripting Tutorial"
    - rlink: https://learn.microsoft.com/en-us/windows/wsl/install
      rtitle: "How to install Linux on Windows with WSL"
      
tags:
  - shellscripting

---

In this tutorial, we'll create a Bash script to monitor and alert about increases in CPU usage on a Linux system. We'll break down the process into simple steps, making it easy to understand even for beginners in Bash scripting.   You will write a script that calculates the average CPU usage over all cores and checks if there's an increase beyond a predefined threshold compared to the previous minute.

## Prerequisites
1. Access to a Linux system with standard command-line tools installed (Windows users can install the [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install) for access to these tools).
2. Basic familiarity with the Linux command line and Bash scripting, or completion of [this Bash Scripting tutorial](https://linuxconfig.org/bash-scripting-tutorial).

## Step 1: Creating a Shell Script

Create a new Bash script file: 

```
nano cpu_monitor.sh
```

And add the shebang line at the top of the file, which tells the system that the script should be passed to the `/bin/bash` program for executing (as if the user had typed these commands themselves):

```
#!/bin/bash
```

## Step 2: Declaring Variables

Declare variables to store previous CPU usage and the threshold for alerting:

```
previous_cpu_usage=0
threshold=10  # You can modify this threshold as needed
```

## Step 3: Writing the Monitoring Loop

Use a `while` loop to continuously monitor the CPU usage:

```
while true; do
    # Script contents will go here
    sleep 10  # Wait for 10 seconds
done
```

## Step 4: Fetching CPU Usage Data

Now, let's write the inside body of the loop above.  **These commands go inside the while loop we've just made.**

We'll use the `top` command to get CPU usage data and `awk` for processing.  This sequence of commands is slightly different for Windows/Linux and Mac Users.

### Windows and Linux Users

```
current_cpu_usage=$(top -bn1 | grep '^%Cpu' | awk '{total += $2 + $4} END {print total/NR}')
```

This command does the following:

* `top -bn1`: Runs top in batch mode for one iteration to get the latest CPU data.
* `grep '^%Cpu'`: Filters lines starting with %Cpu.
* `awk '{total += $2 + $4} END {print total/NR}'`: Sums the user and system usage for each CPU and calculates the average.

### Mac Users

```
current_cpu_usage=$(top -l 1 -n 0 | awk '/CPU usage/ {user+=$3; sys+=$5} END {print (user+sys)/2}' | sed 's/%//g')
```

This command does the following:

* `top -l 1 -n 0`: Runs top in one iteration to get the latest CPU data.
* `awk '/CPU usage/ {user+=$3; sys+=$5} END {print (user+sys)/2}'`: Sums the user and system usage for each CPU and calculates the average. 
* `sed 's/%//g'`: Removes the percentage sign from the output.
  
## Step 5: Comparing CPU Usage and Alerting

Add an `if` statement to compare the current CPU usage with the previous and alert if the increase is beyond the threshold (we'll print the CPU usage otherwise):

```
if [ $(echo "${current_cpu_usage} > ${previous_cpu_usage} + ${threshold}" | bc) -eq 1 ]; then
    echo "Warning: Average CPU usage of ${current_cpu_usage} increased by more than $threshold% in the last minute!"
else
    echo "Average CPU Usage of ${current_cpu_usage} is below the threshold"
fi

previous_cpu_usage=${current_cpu_usage}
```

Here, the `echo` command outputs the variable `current_cpu_usage > previous_cpu_usage + threshold` and pipes that string to the `bc` tool.  To get the value of a variable (rather than using the name of the variable in the string), we enclose the variable name in the `${}` operator.  `bc` is a command line calculator, so it adds the variables together, and does a greater than comparison on the result.  It outputs `1` if it is true.  By putting the expression inside the `$()` operator, we treat the result of the command as a value.  With that value, we do an integer comparison to compare the result to `1` using the `-eq` operator.

## Step 6: Running the Script

To quit nano, press Control-X on your keyboard.  You'll be prompted to save the file you're working on, and you can say yes and hit enter to keep the same filename.  Alternatively, you can press Control-O anytime to save your file.

To run the script, first make it executable:

```
chmod a+x cpu_monitor.sh
```

Then, execute it:

```
./cpu_monitor.sh
```

## Submitting Your Work

You can zip up your file(s) to submit them using a zip program of your choice.
