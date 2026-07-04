---
layout: assignment
permalink: Assignments/FAT
title: "CS376: Operating Systems - FAT File System"

info:
  coursenum: CS376
  points: 100
  goals:
    - Develop a comprehensive understanding of the FAT12 file system, encompassing its structure, organization, and key components, including reserved sectors, FATs, and the root directory.
    - Apply analytical skills to interact with a file system at the binary level, involving the calculation of offsets, interpretation of binary data, and utilization of tools like hexdump for scrutinizing directory entries.
    - Demonstrate the ability to create functional programs capable of enumerating all files within a FAT12 disk image and extracting their contents to the local filesystem.
    - Employ principles of code modularity and best practices in program development, encompassing the creation of structured helper functions, adherence to coding standards, and the efficient handling of data.
    - Apply problem-solving skills and critical thinking to address challenges associated with little-endian representation, parsing 12-bit entries, and potential exploration of advanced tasks such as subdirectory support, while critically evaluating program design and performance.

  rubric:
    - weight: 30
      description: Program Correctness
      preemerging: Code does not compile or execute, or produces critical errors.
      beginning: Code compiles and executes but contains significant logic errors, leading to incorrect results.
      progressing: Code compiles, executes, and produces correct results in most cases but may have minor issues.
      proficient: Code is correct, produces accurate results, and handles edge cases effectively.
    - weight: 20
      description: Listing the FAT File Entries
      preemerging: No evidence of listing FAT file entries.
      beginning: Partial listing of FAT file entries with major issues or inaccuracies.
      progressing: Successfully lists FAT file entries with minor issues or improvements needed.
      proficient: Accurately lists all FAT file entries with clear and well-organized output.
    - weight: 20
      description: "Capstone Task (choose one): Undelete, fsck Chain Verifier, or Defragmenter"
      preemerging: No capstone task is attempted, or the chosen task does not run.
      beginning: A capstone task is attempted but does not correctly follow the FAT cluster chain (e.g., ignores end-of-chain markers or the 12-bit packing).
      progressing: The chosen capstone task follows the cluster chain and produces correct results on the sample image, with minor gaps in edge cases (fragmented files, empty files, or the final partial cluster).
      proficient: The chosen capstone task correctly follows the cluster chain, handles end-of-chain and edge cases, and its output is verified against hexdump with the chain arithmetic explained in the README.
    - weight: 15
      description: Makefile (make, make run, make test, make clean)
      preemerging: Absence of a Makefile, or the Makefile does not compile the program.
      beginning: A Makefile is provided but is missing required targets (run, test, or clean) or fails to build.
      progressing: A Makefile that builds and provides most required targets, but one target is missing or does not behave as described.
      proficient: A complete Makefile providing make, make run, make test, and make clean, each behaving as documented, automating compilation, testing, and cleanup.
    - weight: 15
      description: README
      preemerging: Absence of a README or README lacks essential information.
      beginning: Partial README with significant gaps in explaining how to run the software and its development.
      progressing: Complete README with clear instructions on running the software and its development process.
      proficient: An exemplary README that provides comprehensive guidance on software execution, development, and design decisions.

  readings:
    - rlink: https://web.archive.org/web/20070524053621/https://alumnus.caltech.edu/~pje/dosfiles.html
      rtitle: "A description of the DOS File System"
    - rlink: https://web.archive.org/web/20110118073712/http://www.patersontech.com/Dos/Byte/InsideDos.htm
      rtitle: "An Inside Look at MS-DOS"
    - rlink: https://web.archive.org/web/20080212120457/http://home.no.net/tkos/info/fat.html
      rtitle: "File Allocation Table - How it Seems to Work"
    - rlink: https://www.win.tue.nl/%7Eaeb/linux/fs/fat/fat-1.html
      rtitle: FAT
    - rlink: https://www.ntfs.com/fat_systems.htm
      rtitle: "FAT File Systems.  FAT32, FAT16, FAT12"
    - rlink: https://web.archive.org/web/20080306063127/http://www.exegesis.uklinux.net/gandalf/encrypt/disk.htm
      rtitle: "Disk Structure"
    - rlink: https://www.cs.drexel.edu/~johnsojr/2012-13/fall/cs370/resources/UnderstandingFAT12.pdf
      rtitle: "Understanding FAT12"
    - rlink: https://www.tavi.co.uk/phobos/fat.html
      rtitle: "A Tutorial on the FAT File System"

tags:
  - filesystems
  - fat

---

> **Core Concepts — why this matters.** This assignment reinforces the essential OS topics of *file-system on-disk layout*, *the File Allocation Table as a linked list of clusters*, and *binary data parsing with endianness*. The FAT is the historical ancestor of every file system's block-allocation scheme; understanding how a file's data is scattered across clusters and chained together in a table is exactly the intuition you need for modern file systems, and for the kernel file structures in the [Kernel Data Structures reference](../KernelDataStructures#part-3-reading-file-data-structures).

We have been asked to recover data from floppies that use an old version of Microsoft's DOS file system called FAT12 (i.e. does not have support for long file names, so you can consider all files to have names of up to 8 characters with a 3 character extension). The entire contents of these floppies have been extracted and are stored as separate Unix files.
However, the data files within each floppy volume are still stored within the Unix file in the format used by the MSDOS file system.

You will read the MSDOS FAT12 file system specification so that you can build the following:

1. **(Required)** List all the files on the disk image (`msdosdir`).
2. **(Required — choose ONE) A capstone task** that follows the FAT cluster chain: an **undelete** tool, an **fsck-style chain verifier**, or a **defragmenter**. These are described in the [Capstone Task](#capstone-task-choose-one) section below.
3. **(Extra Credit)** Extract the contents of all files to your local filesystem (`msdosextr`). This is now optional extra credit — pursue it if you finish the required work and want to go further.

> **This is a UDL choice point.** The three capstone options exercise the same core skill — walking a file's chain of clusters through the FAT — but let you pick the framing that motivates you most (data recovery, integrity checking, or performance). Choose the **one** that appeals to you; you are not expected to do more than one.

# Listing the File Entries

For example, assuming that the floppy image was named [samplefat.bin](../files/asmt-fat/samplefat.bin), we should be able to do the following:

```
% msdosdir samplefat.bin
 Volume name is DISK      2
 Volume Serial Number is 16E0-1E1F

ADDNAME  EX_        14,032 08-31-94  12:00a
AVEXTRA  TXT           399 02-28-94  10:19a
DNR      EX_        17,336 12-16-94   6:47p
EMSBFR   EX_         1,407 08-31-94  12:00a
HOSTS                  715 08-31-94   7:37p
IPCONFIG EX_         8,509 08-31-94  12:00a
LICENSE  TXT         2,925 03-28-95   5:23p
LMHOSTS                817 08-31-94   7:36p
NEMM     DO_         1,764 08-31-94  12:00a
NETBIND  COM         8,513 08-31-94  12:00a
NETWORKS               395 08-31-94   6:52p
NMTSR    EX_        12,434 08-31-94  12:00a
PING     EX_        47,277 08-31-94  12:00a
PROTOCOL               795 08-31-94   6:52p
SERVICES             5,973 05-08-95   2:34p
SOCKETS  EX_        27,497 09-01-94   1:22p
TCPDRV   DO_         2,810 08-31-94  12:00a
TCPTSR   EX_        48,433 08-31-94  12:00a
TCPUTILS INI           233 08-31-94  12:00a
TINYRFC  EX_        23,561 12-01-94   7:39p
UMB      CO_         2,353 08-31-94  12:00a
VBAPI    386         9,524 08-31-94  12:00a
VSOCKETS 386         9,535 08-31-94  12:00a
WINSOCK  DL_        25,236 01-23-95   3:21p
WIN_SOCK DL_        16,122 08-31-94  12:00a
WSAHDAPP EX_         3,271 08-31-94  12:00a
WSOCKETS DL_        15,862 08-31-94  12:00a
       27 file(s)        307,728 bytes
```

```
% msdosextr samplefat.bin
Extracting: 27 files
ADDNAME.EX_, AVEXTRA.TXT, DNR.EX_, EMSBFR.EX_, HOSTS, IPCONFIG.EX_,
LICENSE.TXT, LMHOSTS, NEMM.DO_, NETBIND.COM, NETWORKS, NMTSR.EX_,
PING.EX_, PROTOCOL, SERVICES, SOCKETS.EX_, TCPDRV.DO_, TCPTSR.EX_,
TCPUTILS.INI, TINYRFC.EX_, UMB.CO_, VBAPI.386, VSOCKETS.386,
WINSOCK.DL_, WIN_SOCK.DL_, WSAHDAPP.EX_, WSOCKETS.DL_.
```

```
% ls
ADDNAME.EX_   IPCONFIG.EX_  NETWORKS      SOCKETS.EX_   UMB.CO_       WSAHDAPP.EX_
AVEXTRA.TXT   LICENSE.TXT   NMTSR.EX_     TCPDRV.DO_    VBAPI.386     WSOCKETS.DL_
DNR.EX_       LMHOSTS       PING.EX_      TCPTSR.EX_    VSOCKETS.386  sample.flp
EMSBFR.EX_    NEMM.DO_      PROTOCOL      TCPUTILS.INI  WINSOCK.DL_
HOSTS         NETBIND.COM   SERVICES      TINYRFC.EX_   WIN_SOCK.DL_
%
```

## What to Do

Do not begin by writing code to actually read the binary file system dump. Instead, consider the data structures required to implement such a program. In other words, think about your design first (but you knew that!). To give you an idea of what I'm talking about, I have provided one such header file for the boot sector of the disk, which you are not required to use. This header was designed by inspecting the various fields in this sector of the disk, and having fields in the data structure that correspond to fields in the sector. Thus, populating this data structure should be as easy as reading the file byte for byte at that sector, and populating the fields (which can then be printed, etc.). Note that you don't need everything that is defined in this structure; it's just there for illustration. The int imagestream is a file handle to the fat dump so that the file can be opened, read, etc. right from this structure for convenience.

### Boot Sector Data Structure

The boot sector is defined as follows:

```
/**
 * BootStrapSector is the first 512 bytes of the FAT.
 * Two byte fields are little-endian
 *
 * The format of this sector is:
 * byte(s) contents
 * ------- -------------------------------------------------------
 *  0-2 first instruction of bootstrap routine
 *  3-10 OEM name
 *  11-12 number of bytes per sector
 *  13 number of sectors per cluster
 *  14-15 number of reserved sectors
 *  16 number of copies of the file allocation table
 *  17-18 number of entries in root directory
 *  19-20 total number of sectors
 *  21 media descriptor byte
 *  22-23 number of sectors in each copy of file allocation table
 *  24-25 number of sectors per track
 *  26-27 number of sides
 *  28-29 number of hidden sectors
 *  30-509 bootstrap routine and partition information
 *  510 hexadecimal 55
 *  511 hexadecimal AA
 */
 
#ifndef BOOTSTRAP_SECTOR_H
#define BOOTSTRAP_SECTOR_H

typedef unsigned char BYTE; // BYTE is just an unsigned char, a single byte numeric value

struct BootStrapSector {
    BYTE firstInstruction[3];  // This is often a jump instruction to the boot sector code itself
    BYTE OEM[8];  		
    BYTE numBytesPerSector[2];
    BYTE numSectorsPerCluster[1];
    BYTE numReservedSectors[2];
    BYTE numCopiesFAT[1];
    BYTE numEntriesRootDir[2];
    BYTE numSectors[2];
    BYTE mediaDescriptor[1];
    BYTE numSectorsInFAT[2];
    BYTE numSectorsPerTrack[2];
    BYTE numSides[2];
    BYTE numHiddenSectors[2];
    BYTE bootcode[480]; // may include additional metadata
    BYTE signature[2];	// the last bytes of the boot sector are, by definition, 55 AA.  This is a sanity check.
};

#endif
```

Begin by saving this into a `BootStrapSector.h` header file, and you can `#include "BootStrapSector.h"` from your `main.c` program file.

If you `fopen` the disk image file, `malloc` a `BootStrapSector`, and `fread` from the disk image into this data structure, you'll read all these values automatically!  To convert a single `BYTE` to an integer, simply cast it to an `int`.  If you have a 2 byte value, you can convert it to its corresponding integer value by shifting the upper byte and adding the lower byte; for example:

```c
unsigned int result = (unsigned int)(sector.numSectorsInFAT[1] << 8) | (unsigned int)(sector.numSectorsInFAT[0]);
```

For single byte values, you can simply set `unsigned int result = (unsigned int)(sector.numSectorsInFAT[0]);`.   You can print an `unsigned int` using the `%u` placeholder to `printf`.

If you'd like to print a string, like the OEM value, you can pass the `BYTE` array to a function like this, which returns a null-terminated `char*` containing the contents of your byte array that you can print as normal:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * Copies the contents of a char array into a newly allocated, null-terminated char*.
 *
 * @param src The source character array.
 * @param size The number of characters in the source array to copy.
 * @return A pointer to the newly allocated, null-terminated character array.
 *         Returns NULL if memory allocation fails.
 */
char* copyToNullTerminatedCharPtr(const char src[], size_t size) {
    // Allocate memory for the characters plus a null terminator
    char* dest = (char*)malloc((size + 1) * sizeof(char));
    if (dest == NULL) {
        // Handle memory allocation failure
        fprintf(stderr, "Memory allocation failed\n");
        return NULL;
    }

    // Copy the characters from src to dest
    memcpy(dest, src, size);

    // Null-terminate the string
    dest[size] = '\0';

    return dest;
}
```

### Root Directory Data Structure

After the boot sector is the FAT.  You know how many sectors are in the FAT from the values above.  You also know how many copies of the FAT there are, and how many bytes are in each sector.  Multiplying these values together, you can find the offset of the root directory.  Specifically, this formula will give you the byte offset, with which you can `fseek` and `fread` root directory entries.

```c
unsigned int byte_offset_of_root_dir = (num_reserved_sectors + (num_copies_FAT * num_sectors_in_FAT)) * num_bytes_per_sector;
```

You can obtain these integer values from the boot sector fields above by casting and converting the `BYTE` values of these fields to `int`.  

Calculate this offset, `fseek` to this offset within the file from the beginning of the file (`SEEK_SET`), create a data structure for the root directory, and do a loop of `fread` calls into that following data structure.  You know how many root directory entries there are from the boot sector values above as well, so you can loop the appropriate number of times to do these reads.  You can use the following reference to create the data structure like we did above:

```c
/*
From https://www.cs.drexel.edu/~johnsojr/2012-13/fall/cs370/resources/UnderstandingFAT12.pdf
Offset Length Description
0x00 8 Filename
0x08 3 Extension
0x0B 1 Bit field for attributes
0x0C 10 Reserved
0x16 2 Time (coded as Hour*2048+Min*32+Sec/2)
0x18 2 Date (coded as Year-1980)*512+Month*32+Day)
0x1A 2 Starting Cluster Number
0x1C 4 File size (in bytes)
*/
```

Note that some entries may be empty; you can restrict yourself to only the directory entry if `dir->filename[0] >= 'A' && dir->filename[0] <= 'Z'` to skip these empty entries.

#### Converting the Date and Time Fields

The following C program will manipulate the bites of the date and time fields to extract the file date and time.  This example prints each to the screen; you will want to break this into separate functions for the date and the time that returns the result as a `char*` that you will malloc and populate with `sprintf`.

```c
#include <stdio.h>
#include <stdint.h>

typedef unsigned char BYTE

void decodeFATDateTimeFromCharArray(BYTE time[2], BYTE date[2]) {
    // Convert char arrays to uint16_t, assuming little-endian format
    uint16_t encodedTime = (uint16_t)((unsigned char)time[1] << 8 | (unsigned char)time[0]);
    uint16_t encodedDate = (uint16_t)((unsigned char)date[1] << 8 | (unsigned char)date[0]);

    // Decoding time
    uint8_t hour = encodedTime >> 11; // Shift right 11 bits to get the first 5 bits
    uint8_t minute = (encodedTime >> 5) & 0x3F; // Shift right 5 bits and mask to get the next 6 bits
    uint8_t second = (encodedTime & 0x1F) * 2; // Mask to get the last 5 bits and then multiply by 2

    // Decoding date
    uint16_t year = (encodedDate >> 9) + 1980; // Shift right 9 bits to get the first 7 bits then add 1980
    uint8_t month = (encodedDate >> 5) & 0x0F; // Shift right 5 bits and mask to get the next 4 bits
    uint8_t day = encodedDate & 0x1F; // Mask to get the last 5 bits

    // Print decoded date and time
    printf("Decoded Date: %d-%02d-%02d\n", year, month, day);
    printf("Decoded Time: %02d:%02d:%02d\n", hour, minute, second);
}

int main() {
    // Example usage with char arrays:
    BYTE exampleTime[2] = {0x6f, 0x70}; // 14:03:30 in encoded form
    BYTE exampleDate[2] = {0x2f, 0x50}; // 2020-01-15 in encoded form

    decodeFATDateTimeFromCharArray(exampleTime, exampleDate);

    return 0;
}
```

#### Converting the File Size

You can convert the file size to an integer by shifting each byte into place in a 32 bit value, and compute the bitwise or of those bytes:

```c
unsigned int fileSize = 0;
fileSize |= (unsigned int)sizeArray[0];   // Least significant byte
fileSize |= (unsigned int)sizeArray[1] << 8;
fileSize |= (unsigned int)sizeArray[2] << 16;
fileSize |= (unsigned int)sizeArray[3] << 24;  // Most significant byte
```

# Following the FAT Cluster Chain

Both the required capstone task and the extra-credit extraction rely on the same core skill: starting from a file's **starting cluster number** (from its directory entry) and walking the chain of clusters in the File Allocation Table until you hit the end-of-chain marker. This section explains how the FAT stores that chain. Read it carefully — the [Capstone Task](#capstone-task-choose-one) below builds directly on it.

## The 12 in FAT12
There are a few silly nuances in the way the data is represented that you'll want to make sure you're aware of. Particularly, FAT12 stores things in 1.5 bytes, and you'll need to shift and parse accordingly. 

Versions of DOS before 3.00 used a file allocation table with 12-bit entries. Each group of three consecutive bytes contains two 12-bit entries, arranged as follows:

1. The first byte contains the eight least significant bits of the first entry.
2. The four least significant bits of the second byte contain the four most significant bits of the first
entry.
3. The four most significant bits of the second byte contain the four least significant bits of the
second entry.
4. The third byte contains the eight most significant bits of the second entry.
In other words, if UV, WX and YZ are the hexadecimal representations of the three consecutive bytes,
then the entries are XUV and YZW, respectively.

Write a function to help you with this, given a base address!

## Simplifying Assumptions

It is not necessary to code this for a general case FAT filesystem! We're looking to keep it simple for this assignment. Notice that there are no subdirectories on the filesystem you're provided. You don't have to program for subdirectories, etc. (but I'll reward you if you do!). Just get me a program that lists the files in the FAT and extracts them to disk. Don't totally hack it up -- your program should be able to scale to a design that would support subdirectories; however, it is not necessary to actually implement them.

In addition, you can skip files that are marked as deleted.  It would be fun to print these out in your list program, though!

## Getting Started by Viewing the Image

Feel free to use a hex file viewer like `hexdump` to actually view the disk image and verify what you're reading about in the image. This will help you make sense of "endian" issues (i.e. "why is 512 represented as 00 02?")and how to read the individual bytes.

### Using `hexdump` to Examine Directory Entries in a FAT12 File System

To examine directory entries in a FAT12 file system using `hexdump`, you'll need to determine the location of the directory entry and then use `hexdump` to display the relevant data. Here are the steps:

### What to Do

#### Step 1: Determine Directory Entry Location

1. **Understand FAT12 Structure**: Familiarize yourself with the structure of a FAT12 file system. Directory entries are typically 32 bytes long and are located after reserved sectors, FATs, and the root directory.

2. **Calculate Offset**:
   - Find the starting position of the directory entry within the file system. This requires knowledge of the number of reserved sectors, the size of each FAT, and the number of root directory entries. These values are specified in the boot sector of the FAT12 file system.
   
   - Use the following formula to calculate the offset in bytes:
   
     ```
     Offset = (ReservedSectors + NumberOfFATs * FATSize + RootDirEntries * 32) + EntryNumber * 32
     ```

   - Replace the variables with the appropriate values from the boot sector:
     - `ReservedSectors`: Number of reserved sectors.
     - `NumberOfFATs`: Number of File Allocation Tables (usually 2).
     - `FATSize`: Size of each FAT in sectors.
     - `RootDirEntries`: Number of entries in the root directory.
     - `EntryNumber`: The index of the directory entry you want to access (0 for the first entry, 1 for the second, and so on).

#### Step 2: Use `hexdump` to Display Directory Entry

3. **Run `hexdump`**:
   - Open a terminal and use the `hexdump` command to display the directory entry data.
   - The `-s` option is used to skip to the calculated offset, and the `-n` option specifies the number of bytes to display (usually 32 for a full directory entry).

   ```
   hexdump -s Offset -n 32 -C YourFAT12Image.img
   ```

   Replace `Offset` with the calculated offset value and `YourFAT12Image.img` with the path to your FAT12 file image.

4. **Interpret the Output**:
   - The `hexdump` command will display the directory entry data in hexadecimal format, along with the corresponding ASCII representation.
   - Each byte or group of bytes in the output corresponds to a specific attribute or field of the directory entry.
   - Refer to the FAT12 specification to interpret the values correctly.

By following these steps, you can use `hexdump` to view and analyze directory entries in a FAT12 file system, allowing you to inspect file attributes, names, and other relevant information.

## Suggestions

1. Carefully read the articles and specifications provided on this page!  They are essential to learning the file format specifications you'll need to read the FAT12 image file.  If your data structures are correct, you just need to read sequential parts of the file into these data structures, and operating on them accordingly (i.e. read the file attributs/filename and print to screen, go to the appropriate data cluster, traverse the list, write the data to the disk, truncate that data, etc.).
2. Write lots of helper functions! Have one to read a particular byte (or two bytes, or 12 bits) of data -- and a flag to read it little or big endian. This way your entire program will just consist of small functions that call these helpers.  In particular, I recommend writing the following helpers:
  1. Write a helper function to print out metadata about a file (particularly, to print a line of the msdoslist command)
  2. Write a helper function to print out metadata about the filesystem itself (particularly the header lines when you run msdoslist)
  3. Write a helper function to traverse the clusters for a given file and verify that they are correct (use a hex editor!). In other words, just list the clusters that you want to put together for a particular file before you actually write them, so that you can check your work.
3. Write the file data clusters to disk, and truncate if you write too much (for example, if the last cluster is fragmented and thus does not use the whole cluster).  You can use the `filesize mod clustersize` to determine how many bytes from the last cluster should be written.  An even better solution is to pass this remainder directly to your `write` function when writing the last cluster in the file chain, so you'll write exactly the number of bytes.

Wrap this up in a for loop for each entry found in `msdoslist`, and you have the extract program!


# Capstone Task (Choose One)

After you can list files, choose **one** of the following. All three walk a file's cluster chain through the FAT; they differ only in what they *do* with that chain. Pick the one that motivates you most.

## Option A: Undelete

When DOS deletes a file it does **not** erase the data. It does two things: (1) it replaces the first byte of the directory entry's filename with the marker byte `0xE5`, and (2) it frees the file's FAT chain by zeroing the entries. Your undelete tool recovers a deleted file by:

1. Finding directory entries whose first filename byte is `0xE5` (these are the deleted files the [listing suggestions](#suggestions) told you to *skip* — now you seek them out).
2. Reading the starting cluster from the directory entry (this field usually survives deletion).
3. Reading the file size from the directory entry, and reading that many bytes of **consecutive** clusters starting at the starting cluster (because the FAT chain is gone, assume the file was contiguous — a common and reasonable simplification).
4. Writing the recovered bytes to a new file, prompting the user for a replacement first character of the filename.

## Option B: fsck-Style Chain Verifier

A file-system checker validates that the on-disk structures are consistent. Your verifier walks every file's cluster chain and reports problems:

1. For each file in the root directory, follow its cluster chain from the starting cluster to the end-of-chain marker.
2. Verify that the number of clusters in the chain matches the file size (see the chain-length arithmetic below).
3. Detect **cross-linked** clusters (a cluster that appears in two different files' chains) and **lost** clusters (marked in-use in the FAT but not reachable from any directory entry).
4. Print a report of each file's cluster chain and any inconsistencies found.

## Option C: Defragmenter

A fragmented file occupies clusters that are not consecutive on disk, which slows reads. Your defragmenter reports (and optionally rewrites) files so their clusters are contiguous:

1. For each file, follow its cluster chain and record the list of cluster numbers.
2. A file is **fragmented** if its cluster numbers are not consecutive (e.g., `5, 6, 9, 10` instead of `5, 6, 7, 8`).
3. Report each file's fragmentation (number of non-consecutive jumps in its chain).
4. (Ambitious extension) Rewrite the image so each file's clusters are contiguous, updating both the directory entry's starting cluster and the FAT chain accordingly.

## Worked Example: The FAT Chain Arithmetic (Step by Step)

Every option needs two calculations. Work them with concrete numbers the first time.

**Where does the data region start?** Suppose the boot sector gives you:

* reserved sectors = `1`
* number of FATs = `2`
* sectors per FAT = `9`
* root directory entries = `224`
* bytes per sector = `512`
* sectors per cluster = `1`

Compute the byte offset of the data region (cluster 2, the first data cluster) in numbered micro-steps:

1. Sectors used by reserved area: `1`.
2. Sectors used by both FATs: `2 x 9 = 18`.
3. Sectors used by the root directory: `(224 entries x 32 bytes) / 512 bytes-per-sector = 7168 / 512 = 14` sectors.
4. First data sector = `1 + 18 + 14 = 33`.
5. Byte offset of the data region = `33 x 512 = 16896`.

**Where is cluster N on disk?** Because the first two FAT entries (0 and 1) are reserved, data cluster numbering starts at **2**. The byte offset of cluster `N` is:

```
offset(N) = data_region_start + (N - 2) * sectors_per_cluster * bytes_per_sector
```

Worked with `N = 5`, using the numbers above (`data_region_start = 16896`, `sectors_per_cluster = 1`, `bytes_per_sector = 512`):

1. Clusters past the first data cluster: `N - 2 = 5 - 2 = 3`.
2. Bytes per cluster: `sectors_per_cluster x bytes_per_sector = 1 x 512 = 512`.
3. Offset into the data region: `3 x 512 = 1536`.
4. Absolute byte offset: `16896 + 1536 = 18432`. `fseek` here to read cluster 5.

**How do I get the next cluster in the chain?** Look up the current cluster number in the FAT using the 12-bit unpacking rule described in [The 12 in FAT12](#the-12-in-fat12). The value you read is either the **next** cluster number, or an **end-of-chain** marker (`0xFF8`–`0xFFF` for FAT12), which means "this is the last cluster of the file." So the loop is:

```
cluster = starting_cluster (from the directory entry)
while cluster is not an end-of-chain marker (< 0xFF8):
    read/inspect the data at offset(cluster)
    cluster = FAT_lookup(cluster)   # the 12-bit unpacked value
```

**How many clusters should a file occupy?** From the file size, the expected chain length is:

1. Bytes per cluster = `512` (from above).
2. File size = `1300` bytes (example).
3. Clusters needed = `ceil(1300 / 512) = ceil(2.54) = 3` clusters. The last cluster is only partially used (`1300 - 2*512 = 276` bytes), which is why extraction truncates the final cluster.

## Getting Started

1. `hexdump -C samplefat.bin | less` and locate the boot sector fields — confirm you can read "512" as the bytes `00 02` (little-endian). This proves you understand endianness before you write parsing code.
2. Implement `msdosdir` (listing) first and match its output to the sample above. **Expected checkpoint:** your list shows 27 files with the right sizes.
3. Add a helper that, given a cluster number, returns its byte offset (the arithmetic above) — test it by `hexdump`ing a known cluster and comparing.
4. Add a helper that, given a cluster number, returns the next cluster from the FAT (the 12-bit unpacking) — test it by printing a whole chain and checking it ends at an end-of-chain marker.
5. Only then implement your chosen capstone.

## Common Pitfalls

* **Forgetting clusters start at 2.** The `(N - 2)` term is essential; omitting it reads the wrong cluster.
* **Getting the 12-bit packing backwards.** Re-read [The 12 in FAT12](#the-12-in-fat12): for bytes `UV WX YZ`, the two entries are `XUV` and `YZW`.
* **Ignoring the end-of-chain marker,** so your loop runs off the end. Stop at `>= 0xFF8`.
* **Reading past the file size on the last cluster.** Use `filesize mod clustersize` to truncate the final cluster.
* **Little-endian confusion** when combining two bytes into one integer. `(high << 8) | low`.

## Makefile Requirements

Include a `Makefile` supporting the following standard targets:

| Target       | What it must do                                                                       |
| ------------ | ------------------------------------------------------------------------------------- |
| `make`       | Compile `msdosdir` and your capstone program, with no errors.                          |
| `make run`   | Build if needed and run `msdosdir` on the sample image.                                |
| `make test`  | Build if needed and run your test cases (e.g., listing plus your capstone on the sample image), printing observable pass/fail output. |
| `make clean` | Remove all executables, object files, extracted files, and core dumps so a fresh `make` starts clean. |

## Glossary

* **Cluster** — the smallest unit of disk space a file occupies; a file is a chain of clusters.
* **FAT (File Allocation Table)** — a table with one entry per cluster; each entry is either the *next* cluster in a file's chain or an end-of-chain marker. It is a linked list stored as an array.
* **End-of-chain marker** — a FAT value (`0xFF8`–`0xFFF` in FAT12) meaning "last cluster of this file."
* **Directory entry** — a 32-byte record holding a file's name, attributes, starting cluster, and size.
* **Little-endian** — byte ordering where the least significant byte comes first (why 512 appears as `00 02`).
* **Fragmentation** — the condition of a file whose clusters are not stored consecutively on disk.
* **Cross-linked clusters** — a corruption where one cluster is claimed by two files' chains.
