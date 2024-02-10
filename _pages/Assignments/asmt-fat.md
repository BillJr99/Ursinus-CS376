---
layout: assignment
permalink: Assignments/FAT
title: "CS376: Operating Systems - FAT File System"
excerpt: "CS376: Operating Systems - FAT File System"
excerpt_separator: <!--more-->

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
      description: Extracting Files
      preemerging: No evidence of file extraction.
      beginning: Partial file extraction with major issues or inaccuracies.
      progressing: Successfully extracts files with minor issues or improvements needed.
      proficient: Accurately extracts all files with proper handling of file data and formats.
    - weight: 15
      description: Makefile
      preemerging: Absence of a Makefile or Makefile does not compile or execute both programs.
      beginning: Partially functional Makefile with significant gaps in compiling or testing both programs.
      progressing: Functional Makefile that compiles and tests both programs but may lack automation or efficiency.
      proficient: A complete and efficient Makefile that automates compilation, testing, and cleanup effectively.
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

We have been asked to recover data from floppies that use an old version of Microsoft's DOS file system called FAT12 (i.e. does not have support for long file names, so you can consider all files to have names of up to 8 characters with a 3 character extension). The entire contents of these floppies have been extracted and are stored as separate Unix files.
However, the data files within each floppy volume are still stored within the Unix file in the format used by the MSDOS file system.

You will read the MSDOS FAT12 file system specification so that you can create programs that will be able to write the following **two** programs:

1. List all the files on the disk image
2. Extract the contents of those files to your local filesystem.

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

Note that this structure is written as a `class` in C++.  You are welcome to use either C++ or C for your implementation; if you use C, you can simply convert this to a struct and include the methods in your implementation file.

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

#include "types.h"

class BootStrapSector {
	public:
           BootStrapSector(int is);
           int getNumBytesInFAT();
           int getNumClusters();
           int getNumEntriesInRootDir();
           int getNumBytesInReservedSectors();
           int getNumCopiesFAT();
           int getNumBytesPerCluster();
           BYTE* getVolumeLabel();
           BYTE* getVolumeSerialNumber();
           BYTE* getFormatType();
		
	private:
            void readBootStrapSector();
            
            int imageStream;
            
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
            BYTE formatType[8]; // FAT12 or FAT16 in this program
            BYTE hex55AA[2];	// the last bytes of the boot sector are, by definition, 55 AA.  This is a sanity check.
            BYTE volumeLabel[11];
            BYTE volumeSN[4];
};

#endif
```

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

