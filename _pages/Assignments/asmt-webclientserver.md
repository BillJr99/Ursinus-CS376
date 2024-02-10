---
layout: assignment
permalink: Assignments/WebClientServer
title: "CS376: Operating Systems - Web Client and Server"

info:
  coursenum: CS376
  points: 100
  goals:
    - Explain HTTP request and response structures, including methods, headers, and bodies, through the practical implementation of an HTTP client and server.
    - Use sockets for creating and managing connections between a client and a server.
    - Handle errors in network communication, including interpreting and responding with appropriate HTTP status codes, particularly in server-side operations.
    -  Utilize file input/output operations in a networked environment, such as reading and serving files from a server, reinforcing understanding of file handling in conjunction with network programming.
    
  rubric:
  - weight: 15
    description: Correct Use of Sockets
    preemerging: Unable to establish socket connections.
    beginning: Establishes connections but with frequent errors or instability.
    progressing: Establishes stable connections but lacks optimal usage or efficiency.
    proficient: Demonstrates efficient and error-free socket management.
  - weight: 15
    description: Implementing the Client
    preemerging: Fails to send HTTP requests.
    beginning: Sends HTTP requests but with incorrect format or errors in handling responses.
    progressing: Sends correctly formatted requests and mostly handles responses well.
    proficient: Flawlessly sends requests and handles responses, adhering to HTTP standards.
  - weight: 20
    description: Implementing the Server
    preemerging: Server does not process requests.
    beginning: Server processes requests but struggles with correct file serving or stability.
    progressing: Server handles requests and serves files with minor issues.
    proficient: Server perfectly handles requests and file serving, ensuring stability and efficiency.
  - weight: 10
    description: Providing a Makefile
    preemerging: No Makefile provided.
    beginning: Provides a basic Makefile with limited functionality.
    progressing: Makefile covers essential functions but lacks advanced features or optimization.
    proficient: Comprehensive and optimized Makefile, facilitating seamless compilation and execution.
  - weight: 10
    description: Writing a README
    preemerging: No README or extremely poor documentation.
    beginning: Basic README provided but lacks clarity or completeness.
    progressing: Good README with most necessary instructions and information.
    proficient: Excellent, well-structured README offering clear, comprehensive instructions and insights.
  - weight: 10
    description: Correctly Tokenizing the HTTP Request
    preemerging: Unable to tokenize or parse HTTP requests.
    beginning: Basic tokenization with frequent errors or misinterpretations.
    progressing: Correctly tokenizes most requests but with minor flaws.
    proficient: Flawlessly tokenizes and parses HTTP requests, ensuring accurate processing.
  - weight: 10
    description: Correctly Forming the HTTP Response
    preemerging: Fails to form correct HTTP responses.
    beginning: Forms basic HTTP responses with errors or non-standard structures.
    progressing: Forms mostly correct responses with minor discrepancies.
    proficient: Excellently constructs and sends fully compliant HTTP responses.
  - weight: 10
    description: Handling HTTP Errors
    preemerging: Does not handle or incorrectly handles HTTP errors.
    beginning: Basic handling of some HTTP errors, but lacks comprehensiveness.
    progressing: Good handling of most HTTP errors, with minor oversights.
    proficient: Expertly identifies and handles a wide range of HTTP errors, providing appropriate responses.

  readings:
    - rlink: http://csapp.cs.cmu.edu/2e/ics2/code/netp/echoclient.c
      rtitle: CSAPP Echo Client
    - rlink: https://csapp.cs.cmu.edu/3e/ics3/code/include/csapp.h
      rtitle: "csapp.h"
    - rlink: https://www.cs.cmu.edu/~213/code/22-netprog1/csapp.c
      rtitle: "csapp.c"
    - rlink: https://condor.depaul.edu/glancast/374class/docs/csapp_compile_guide.html
      rtitle: "Compiling with the CSAPP Library"

tags:
  - socket

---

# Assignment: Building an HTTP Client and Server

## Overview

In this assignment, you will develop a basic HTTP client and a corresponding HTTP server. This exercise aims to deepen your understanding of HTTP protocols, network sockets, and the client-server model. By the end of this assignment, you will have a functional understanding of HTTP headers, bodies, and error codes, along with practical experience in handling network communications.

---

## Part 1: HTTP Client

### Objective

Your task is to create an HTTP client that sends a GET request to a specified web server and displays the server's response.

### Requirements

1. **Command-Line Arguments**: Your program should accept command-line arguments for:
   - The hostname of the web server.
   - The page to request (e.g., `/index.html`).
   - The port number (default to 80, but allow custom ports).

2. **Network Sockets**: Utilize network sockets to establish a connection with the server. You can use the CSAPP `open_clientfd()` function for this purpose.

3. **HTTP Request Format**: Construct and send an HTTP GET request in the following format:

```
GET /[page] HTTP/1.1\r\n
Host: [hostname]\r\n
\r\n
```

Ensure that the `[page]` and `[hostname]` placeholders are replaced with the appropriate command-line arguments.

4. **Response Handling**: Read and display the entire response from the server.

5. **Reference**: Use the CSAPP echo client as a guide for handling network communications.

### Example Usage

```bash
$ ./httpClient www.example.com /index.html 80
```

## Part 2: HTTP Server

### Objective

Develop an HTTP server that listens for connections, processes GET requests, and serves the requested files.

### Requirements

1. **Command-Line Argument**: The server should accept a port number as a command-line argument.

2. **Socket Creation and Listening**: Use `socket()` or CSAPP `open_listenfd()` to create a listening socket. The server should continuously listen for incoming connections using `accept()`.

3. **Request Processing**:
   - The server should read HTTP requests of the form `GET /path HTTP/1.1\\r\\n\\r\\n`.
   - Tokenize the request string to extract the HTTP method (verb) and the requested path.
   - Validate the HTTP method and handle the request accordingly.

4. **Response Handling**:
   - If the file exists, read the file and send its contents back to the client.
   - If the file does not exist, send an appropriate HTTP error code, such as `404 Not Found`.

5. **File Operations**: Refer to the CSAPP file read/write examples for handling file descriptors.

6. **Reference**: Use the CSAPP echo server as a template for managing connections and requests.

### Example Usage

```bash
$ ./httpServer 8080
```

### Additional Guidelines
* **Error Handling:** Implement robust error handling for both client and server, including handling network errors and malformed requests.
* **HTTP Headers and Bodies:**
  * **Client:** Ensure proper formatting of the HTTP request headers.
  * **Server:** Include necessary headers in the response, such as Content-Type.
* **HTTP Error Codes:** The server should be able to handle common HTTP errors, such as 404 Not Found.

## Part 3: Makefile

Provide a makefile (or two) to build and run your client and server.
