---
layout: assignment
permalink: Projects/BootingCustomKernel
title: "CS376: Operating Systems - Booting a Custom Linux Kernel"
excerpt: "CS376: Operating Systems - Booting a Custom Linux Kernel"

info:
  coursenum: CS376
  points: 100
  goals:
    - To use qemu to boot a virtualized operating system
    - To build and install a custom Linux kernel
    - To manipulate the source code of the Linux kernel

  rubric:
  - weight: 80
    description: Code Functionality
    preemerging: Test cases or reproduction steps are not given or are not appropriate
    beginning: A reasonable attempt is made with feasible test cases documented in the README
    progressing: Code works and appropriate test cases given in the README pass with minor adjustments 
    proficient: Code works when the patch or source files are applied, and testing succeeds with the appropriate steps given in the README, along with my own test cases
  - weight: 20
    description: Code Documentation and README
    preemerging: Both the README and documentation are lacking
    beginning: Either the README or documentation are lacking
    progressing: A README and code documentation are provided 
    proficient: A README and code documentation are provided and appropriate   

  readings:
    - rlink: https://elixir.bootlin.com/linux/v2.6.22.19/source
      rtitle: The Linux Source Code
    - rlink: https://launchpad.net/linux/2.6.22/2.6.22.19
      rtitle: "Linux Version 2.6.22.19 Download Page"
    - rlink: https://kernelnewbies.org/
      rtitle: "Kernelnewbies - a Community of Aspiring Linux kernel developers"

tags:
  - linux
  - kernel
  - project

---

In this sequence of projects, you will manipulate and modify the Linux Kernel to become familiar with the structures and functionality of the kernel.  The kernel is nothing more than a program that runs at boot time and manages the resources and programs that you will use.  These projects were inspired by SIGCSE 2010, Dr. Michael Haungs of California Polytechnic State University, Robert Hess of Oregon State University, and Carolyna Ayala of Algonquin College.  Thanks also to Gaylord Holder and Keith Horrocks from Drexel University for their work in automating virtual machine creation and updating the directions found here.

## Setting up Your Environment

We have installed Debian Linux onto a virtual machine disk image that contains the software you'll need to complete these projects pre-installed.  It also contains the source code for the Linux kernel that you will modify in the user home directory.  We are using kernel version [2.6.22.19](https://launchpad.net/linux/2.6.22/2.6.22.19), which is fairly old but suitable to explore the algorithms and data structures that are being evolved and optimized today.

### Step 1 - Installing QEMU
Begin by installing QEMU on your local computer, following the instructions for your operating system found [on this page](https://www.qemu.org/download/).  Windows users can install a pre-built binary installation of QEMU [from here](https://qemu.weilnetz.de/w64/); however, installations that include a hypervisor such as [kvm](https://www.tecmint.com/install-qemu-kvm-ubuntu-create-virtual-machines/) will run faster.  

### Step 2 - Downloading the Disk Image

You can download the base installation image from the multi-part zip file below.  Download each of the 6 image parts, and open and extract the `001` file in the zip program of your choice.  This will extract the `base.qcow` file.

* [base.zip.001](../files/proj-booting/base.zip.001)
* [base.zip.002](../files/proj-booting/base.zip.002)
* [base.zip.003](../files/proj-booting/base.zip.003)
* [base.zip.004](../files/proj-booting/base.zip.004)
* [base.zip.005](../files/proj-booting/base.zip.005)
* [base.zip.006](../files/proj-booting/base.zip.006)

### Step 3 - Creating Your Own Disk Image

You could work directly with this disk image.  However, if you make a mistake, it would be helpful to return to that image without re-downloading it.  Therefore, I recommend creating a snapshot from this image, which will create a much smaller second disk image connected to the first that contains only your revisions.

To create your disk image with a base image of the `base.qcow2` file you just downloaded, run the following command:

`qemu-img create -f qcow2 -b base.qcow2 -F qcow2 local.qcow2`

Note that you must keep these files together in your working directory.

### Step 4 - Booting Your Disk Image

To boot your image, run the following command:

`qemu-system-x86_64 -drive file=local.qcow2 -device e1000,netdev=net0 -netdev user,id=net0 -m 1024M -smp 8 -accel tcg`

There are several parts here:

* `qemu-system-x86_64`: This is the qemu client in which your virtual machine will boot.  We are booting a 64-bit virtual machine, so we run the client for `x86_64` architectures.
* `-drive file=local.qcow2`: This points to the virtual disk you just created (which is automatically linked to `base.qcow2` from earlier).
* `-device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::2222-:22`: This creates a virtual network device within your virtual machine that will be connected through your host computer's network connection.  It also forwards port 2222 to the ssh port of 22, so that you can remotely log into the virtual machine.  I've omitted the `hostfwd` parameter above in case you do not plan to use ssh or scp to log in and/or copy files to your virtual machine, but you are welcome to add it as shown here.
* `-m 1024M -smp 8 -accel tcg`: These specify how much memory, how many CPU cores, and the underlying virtualization accelerator to use, respectively.

Your virtual machine should boot and allow you to log in.  There are two options for logging into your machine:

1. Log in as `user` with password `user`
2. Log in as the root user `root` with password `root`

Please log into each of these accounts and change the password to better ones that you will remember right away.  You can run the `passwd` command to change your password.

### Optional - Logging into Your Virtual Machine with ssh

If you'd like to use tools like `ssh` or `scp` to remote login to your virtual machine, you can enable the `hostfwd` parameter above, and run the following command from your local computer:

`ssh localhost -p 2222 -o PasswordAuthentication=yes -l user -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa`

## Compiling Your Custom Kernel

In the `user` account home directory, you will see a `linux-2.6.22-19` directory and a `pristine_linux` directory, which contain the Linux Kernel source code.  It is included twice so that you have a convenient backup of the original source code.

You could work directly in the `linux-2.6.22-19` directory; however, it is somewhat slow and cumbersome to work within the virtual machine.  Instead, I recommend cloning the source code from a git repository on your local computer **and** on the virtual machine.  As you work from your local computer, you can push to the repository, and pull from that repository on the virtual machine to build.

For now, we'll just build and modify the kernel directly on the virtual machine, to make sure everything is set up correctly.

### Step 1 - Cloning the Kernel Source from git

Rename the `linux-2.6.22.19` directory to something else like `linux-2.6.22.19-orig` so that it is out of the way of the repository you're about to clone.  

Navigate to [https://github.com/BillJr99/linux-2.6.22.19](https://github.com/BillJr99/linux-2.6.22.19) and click the `Fork` button to copy the repository into your own account.

Clone that forked repository on the virtual machine with the following command:

`git clone https://github.com/<your git username>/linux-2.6.22.19.git`

### Step 2 - Building the Kernel

Now it's time to compile the kernel!  The kernel is just another piece of software, and it compiles and executes almost like any other.  The major exception is that it runs immediately upon bootup, rather than executing it from a console.  As a result, it runs with supervisory privileges over the hardware of the computer not enjoyed by any other user software.

First, we'll download a custom Makefile and build configuration file.  Normally, you would run `make menuconfig` to make choices about the hardware on your computer and what preferences and options you wish to set for this kernel.  This can be a cumbersome process, so I've set these options for you already and made them available for you to use.  Re-initialize the settings for this kernel and download my configuration settings into your kernel source directory with the following commands:

```
cd linux-2.6.22-19
make clean; make mrproper; make clean
wget --no-check-certificate www.billmongan.com/Ursinus-CS376/files/proj-booting/config-2.6.22.19-debian -O .config
wget --no-check-certificate www.billmongan.com/Ursinus-CS376/files/proj-booting/Makefile -O Makefile`
make oldconfig
```

Copying my `.config` and `Makefile` files allows us to configure a minimal kerenel without unnecessary device driver modules and gcc stack overflow protections, which will speed up compilaion.

To build the kernel, you run `make`.  You should do so within your linux directory (i.e., `linux-2.6.22.19`), and be sure to type EXTRAVERSION='.19-LASTNAME' -- substituting your last names for LASTNAME (no spaces!).  This way, your kernel will show up in the boot menu of your virtual machine with your name on it (along with the one that came with the operating system), so you'll know which one is which.

This will build a file called `arch/x86_64/boot/bzImage`.  

### Step 3 - Installing Your Kernel to the Boot Loader

There is a bootloader program called GRUB that lists and boots the kernel of your choice, which is useful if you have more than one kernel or more than one Operating System installed.  To add this new image file to your `/boot` directory and to GRUB, execute the following commands (starting with `su`, which gives you supervisor / root privileges to make these changes):

```
su
make install
update-grub
```

You can exit the root environment back to your user account by entering the `exit` command.  You'll see the shell prompt switch from a `#` to a `$`, indicating you've switched from root (`#`) to user (`$`) mode.

## Booting Your Kernel

You can boot the kernel on your virtual machine by typing the same command to the one you used to boot the operating system earlier:

`qemu-system-x86_64 -drive file=local.qcow2 -device e1000,netdev=net0 -netdev user,id=net0 -m 1024M -smp 8 -accel tcg`

Alternatively, if you are still in your virtual machine, you can simply reboot it:

```
su
reboot
```

In the boot menu, choose the kernel with your name on it!  Welcome to your custom kernel!

## Making Your First Kernel Modification

To get your feet wet, you'll make a small change to the kernel source tree.  Specifically, you'll print a message at boot time if (and only if) a kernel command line argument was passed.

### Step 1 - Modifying the Kernel

Specifically, if the new printme argument is set, you'll print "Hello World from Me!" to the screen during boot time.  Here's what to do:

* `cd linux-2.6.22.19`
* `cd init`
* `vim main.c`
* Look at how kernel command line `reset_devices` is coded. This provides an example of how you should code the `printme` kernel parameter.  Search for `reset_devices` everywhere in this file by typing `/reset_devices` and pressing the `n` character to cycle through the next instance of the string.  Everywhere you find `reset_devices`, copy the block and change `reset_devices` to `printme`.
* After the `calibrate_delay()` call in the `start_kernel()` function, add code to check for the `printme` parameter by checking if `printme` is equal to `1`, and printk("Hello World from Me!\n"); if it is set.
* Save and exit

### Step 2 - Building and Installing the Kernel

Build and boot the kernel as before:

* `cd ~/linux-2.6.22.19`
* `make EXTRAVERSION='.19-LASTNAME' -j8
* `su`
* `make install`
* `update-grub`
* `reboot`

When the kernel reboots, arrow down to your kernel but don't press enter just yet.  Press `e` instead to edit the boot parameters.  Arrow down to the bottom line and to the far right of that line.  Insert the word `printme` there (with a space after whatever came before).  Continue booting by pressing `Control-X`.  Note that you'll normally be able to simply press enter to boot your kernel, since we won't do much with boot parameters from here on out.

### Step 3 - Verifying the Results

When the virtual machine boots, log in and run the following command to see if your log message was printed:

`dmesg | grep "Hello World from Me"`

Then, boot normally without adding this `printme` parameter (by hitting enter in GRUB when you select your kernel), and re-run this `dmesg | grep "Hello World from Me"`.  The message should only appear when you include the `printme` parameter!

## Creating a diff Patch for Submission

The kernel is a large source tree.  As a result, it is helpful to create a single file that captures the revisions you've made.

Commit and push your changes to git as usual with the following commands:

```git add init/main.c
git commit -m "Adding a printme boot parameter that prints to the screen"
git pull
git push
```

You can see a list of your commits by typing `git log`.  Each log entry will have a hexadecimal hash value that identifies the commit.  You can create a diff patch between two such commits by copying the hashes (I'll call them `firstcommit` and `lastcommit` here), and executing:

`git format-patch firstcommit^..lastcommit --stdout >my.patch`

Submit this file as part of your submission.

## Note - For KVM Users

If you have kvm installed, you can replace this command:

`qemu-system-x86_64 -drive file=local.qcow2 -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::2222-:22 -m 1024M -smp 8 -accel tcg`

with this one:

`kvm -curses -drive file=local.qcow2 -redir tcp:2222::22`

In addition, you can boot your kernel directly from your local computer (without having to clone, build, run `make install` and run `update-grub` on the virtual machine) by adding these two parameters to the `kvm` command:

`-kernel arch/x86_64/boot/bzImage -append 'root=/dev/hda1 ro'`

You can add the `printme` parameter right after `root=/dev/hda1 ro'` to add that kernel parameter at boot time as well.

<!-- 
References:
https://vault.centos.org/5.5/isos/i386/
https://qemu.weilnetz.de/w64/
https://superuser.com/questions/986875/grub2-is-it-possible-to-disable-graphics-drivers-through-kernel-boot-command
https://wiki.qemu.org/Documentation/Networking
https://support.genymotion.com/hc/en-us/articles/9500420360093-I-get-the-error-no-matching-host-key-type-found-Their-offer-ssh-rsa-when-trying-to-connect-with-SSH
https://launchpad.net/linux/2.6.22/2.6.22.19
https://www.qemu.org/download/
Linux Cross Reference: https://elixir.bootlin.com/linux/v2.6.22.19/source
-->