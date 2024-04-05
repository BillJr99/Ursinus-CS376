---
layout: assignment
permalink: Projects/BootingCustomKernel
title: "CS376: Operating Systems - Booting a Custom Linux Kernel"

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
      rtitle: "Kernelnewbies - a Community of Aspiring Linux Kernel Developers"
    - rlink: https://makelinux.github.io/kernel/map/
      rtitle: "Interactive Map of Linux Kernel Functions"
    - rlink: https://tldp.org/
      rtitle: "The Linux Documentation Project"

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

Windows users may find it more convenient to add their qemu installation directory (likely `C:\Program Files\qemu`) to their system path.  You can do so by following [these instructions](https://www.c-sharpcorner.com/article/how-to-addedit-path-environment-variable-in-windows-11/).

### Step 2 - Downloading the Disk Image

If the image is not already downloaded in your environment, you can download the base installation image from the multi-part zip file below.  Download each of the 6 image parts, and open and extract the `001` file in the zip program of your choice, or concatenate the files together into a single `base.zip` file with a command like `cat base.zip.00? > base.zip`, and extract the resulting file.  This will extract the `base.qcow` file.

* [base.zip.001](../files/proj-booting/base.zip.001)
* [base.zip.002](../files/proj-booting/base.zip.002)
* [base.zip.003](../files/proj-booting/base.zip.003)
* [base.zip.004](../files/proj-booting/base.zip.004)
* [base.zip.005](../files/proj-booting/base.zip.005)
* [base.zip.006](../files/proj-booting/base.zip.006)

On Linux or mac, here is a one-liner that accomplishes all of this:

```
for i in {1..6}; do wget "https://www.billmongan.com/Ursinus-CS376/files/proj-booting/base.zip.00$i"; done; cat base.zip.00{1..6} >base.zip; unzip base.zip
```

### Step 3 - Creating Your Own Disk Image

You could work directly with this disk image.  However, if you make a mistake, it would be helpful to return to that image without re-downloading it.  Therefore, I recommend creating a snapshot from this image, which will create a much smaller second disk image connected to the first that contains only your revisions.

To create your disk image with a base image of the `base.qcow2` file you just downloaded, run the following command:

`qemu-img create -f qcow2 -b base.qcow2 -F qcow2 local.qcow2`

You can include the full path to `base.qcow2` in this command.  For example, you would type `/opt/kernelproj/base.qcow2` in the command above if the file is located there.

### Step 4 - Booting Your Disk Image

To boot your image, run the following command:

`qemu-system-x86_64 -display curses -drive file=local.qcow2 -m 1024M -smp 8 -accel tcg`

There are several parts here:

* `qemu-system-x86_64`: This is the qemu client in which your virtual machine will boot.  We are booting a 64-bit virtual machine, so we run the client for `x86_64` architectures.  
    * **Note**: If you have access to `kvm`, you can replace this word with `kvm` any time it is used.  **It is worth trying this to see if your machine supports it, because it will run much faster this way!**
* `-display curses`: This causes the machine to boot in text mode, which can be a bit faster and more convenient than popping up a window.  It is optional and can be removed if your installation does not support it. 
* `-drive file=local.qcow2`: This points to the virtual disk you just created (which is automatically linked to `base.qcow2` from earlier).
* `-m 1024M -smp 8 -accel tcg`: These specify how much memory, how many CPU cores, and the underlying virtualization accelerator to use, respectively.  These are also optional and may be removed selectively as needed.

### Step 5 - Logging In

Your virtual machine should boot and allow you to log in.  There are two options for logging into your machine:

1. Log in as `user` with password `user`
2. Log in as the root user `root` with password `root`

Please log into each of these accounts and change the password to better ones that you will remember right away.  You can run the `passwd` command to change your password.  

When you are done, you can quit the virtual machine by becoming the root user (either by logging in as root or by typing `su`, pressing enter, and entering the root password), and typing the `halt` command.

## Compiling Your Custom Kernel

Within the virtual machine, in the `user` account home directory, you will see a `linux-2.6.22.19` directory and a `pristine_linux` directory, which contain the Linux Kernel source code.  It is included twice so that you have a convenient backup of the original source code.

You can work directly in the `linux-2.6.22.19` directory; however, it is somewhat slower to work directly within the virtual machine.  I discuss later how to clone and build the kernel on your local computer, and then pass the resulting kernel image to kvm, if you are using it.  Feel free to skip below to **Step 1** to build and modify the kernel directly on the virtual machine, or continue on if you are using `kvm` and want to build locally.

### Optional Step 0 - Building the Kernel Outside of the Virtual Machine and Using git

Some versions of `qemu` and `kvm` enable passing the compiled kernel file from outside of the virtual machine.  This is very convenient because you can develop and compile on your local computer, and that compilation is typically much faster than what it would be on the virtual machine itself.  Because of the way we'll use these tools, it is necessary to work within a Linux environment to do this.  In addition, we'll download the kernel source code from a git repository so that you can commit your changes and share your work more easily.

If you are setting up your own Linux-based computer, you can install the following software packages, and run the following configuration commands:

```
sudo apt install qemu-system qemu-kvm virt-manager virtinst libvirt-clients bridge-utils libvirt-daemon-system build-essential gcc gdb valgrind vim git unzip
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
```

Begin by downloading the kernel source code from git.  Navigate to [https://github.com/BillJr99/linux-2.6.22.19](https://github.com/BillJr99/linux-2.6.22.19) and click the `Fork` button to copy the repository into your own account.  Next, clone that forked repository on your computer with the following command:

```
git clone git@github.com:<your git username>/linux-2.6.22.19.git
```

This kernel uses a rather old Makefile, and so an older verison of Make is needed to build it.  If you are compiling locally, you can download a compatible and prebuilt version of Make for Linux operating systems by downloading [make3.tar.bz2](../files/proj-booting/make3.tar.bz2), and extracting it with:

```
tar xvjpf make3.tar.bz2
```

You can run this version of make by running `./home/wmm24/public/make3/bin/make` wherever you would run make, or run the command:

```
alias make3="~/home/wmm24/public/make3/bin/make"
```

This command allows you to use this version of make every time by entering the command `make3` whenever you are asked to run `make` below.  If this version of make is preinstalled on your environment (for example, at `/opt/make3`), you can execute this command as `alias make3="/opt/make3/bin/make"`.  

Alternatively, you can download a 3.x version of make from the [make website](https://ftp.gnu.org/gnu/make/) and build the software yourself on your local computer architecture.

Now, you can work entirely from your local computer and verify the results on your virtual machine! 

### Step 1 - Building the Kernel

Now it's time to compile the kernel!  The kernel is just another piece of software, and it compiles and executes almost like any other.  The major exception is that it runs immediately upon bootup, rather than executing it from a console.  As a result, it runs with supervisory privileges over the hardware of the computer not enjoyed by any other user software.

First, we'll download a custom Makefile and build configuration file.  Normally, you would run `make menuconfig` to make choices about the hardware on your computer and what preferences and options you wish to set for this kernel.  This can be a cumbersome process, so I've set these options for you already and made them available for you to use.  Re-initialize the settings for this kernel and download my configuration settings into your kernel source directory with the following commands:

```
cd linux-2.6.22.19
make clean; make mrproper; make clean
wget --no-check-certificate www.billmongan.com/Ursinus-CS376/files/proj-booting/config-2.6.22.19-debian -O .config
wget --no-check-certificate www.billmongan.com/Ursinus-CS376/files/proj-booting/Makefile -O Makefile
make oldconfig
```

Copying my `.config` and `Makefile` files allows us to configure a minimal kerenel without unnecessary device driver modules and gcc stack overflow protections, which will speed up compilaion.

To build the kernel, you run `make`.  You should do so within your linux directory (i.e., `linux-2.6.22.19`), and be sure to type `EXTRAVERSION='.19-LASTNAME'` -- substituting your last names for LASTNAME (no spaces!).  This way, your kernel will show up in the boot menu of your virtual machine with your name on it (along with the one that came with the operating system), so you'll know which one is which.  So, you will run this command:

```
make -j2 EXTRAVERSION='.19-LASTNAME'
```

This will build a file called `arch/x86_64/boot/bzImage` using two threads (`j` for "jobs").  

### Step 2 - Installing Your Kernel to the Boot Loader

There is a bootloader program called GRUB that lists and boots the kernel of your choice, which is useful if you have more than one kernel or more than one Operating System installed.  **If you are working outside of the virtual machine, you can skip this step!**

#### Working within the Virtual Machine

If you are working within your virtual machine, you can add this new image file to your `/boot` directory and to GRUB, execute the following commands (starting with `su`, which gives you supervisor / root privileges to make these changes):

```
su
make install
update-grub
```

You can exit the root environment back to your user account by entering the `exit` command.  You'll see the shell prompt switch from a `#` to a `$`, indicating you've switched from root (`#`) to user (`$`) mode.

## Booting Your Kernel

You can boot the kernel on your virtual machine by typing the same `qemu-system-x86_64` or `kvm` command to the one you used to boot the operating system earlier.  You will add a couple of parameters if you are passing the kernel from outside of the virtual machine, an you will select your kernel from the boot menu if you are booting from inside your virtual machine.

### Working within the Virtual Machine

If you are still in your virtual machine, you can simply reboot it:

```
su
reboot
```

In the boot menu, choose the kernel with your name on it!  Welcome to your custom kernel!

### Working outside Your Virtual Machine

If you are working outside of your virtual machine, it is easy to boot the virtual machine with a custom kernel!  Once you have configured and compiled the kernel using the steps above, you can add these two parameters to the `kvm` command you ran earlier to boot your virtual machine: 

```
-kernel linux-2.6.22-19/arch/x86_64/boot/bzImage -append 'root=/dev/hda1 ro'
```

  Note that you may need to run `cd ..` to move back to the directory containing your `local.qcow2` file; otherwise, you'll want to change the paths to your `local.qcow2` file and/or to the `bzImage` file in this command.
  
## Making Your First Kernel Modification

To get your feet wet, you'll make a small change to the kernel source tree.  Specifically, you'll print a message at boot time if (and only if) a kernel command line argument was passed.

### Step 1 - Modifying the Kernel

Specifically, if the new printme argument is set, you'll print `"Hello World from Me!"` to the screen during boot time.  Here's what to do:

* `cd linux-2.6.22.19`
* `cd init`
* `vim main.c`
* Look at how kernel command line `reset_devices` is coded. This provides an example of how you should code the `printme` kernel parameter.  Search for `reset_devices` everywhere in this file by typing `/reset_devices` and pressing the `n` character to cycle through the next instance of the string.  Everywhere you find `reset_devices`, copy the block and change `reset_devices` to `printme`.  Note that there are a couple of instances of this variable, so look closely and replicate the entire block that you find for each instance!
* After the `calibrate_delay()` call in the `start_kernel()` function, add code to check for the `printme` parameter by checking if `printme` is equal to `1`, and `printk("Hello World from Me!\n");` if it is set.
* Save and exit.

### Step 2 - Building and Installing the Kernel

Build and boot the kernel as before:

* `cd ~/linux-2.6.22.19`
* `make EXTRAVERSION='.19-LASTNAME' -j2`

### Working within the Virtual Machine

If you are working within the virtual machine, install your kernel like this:
* `su`
* `make install`
* `update-grub`
* `reboot`

From within the virtual machine, when the kernel reboots, arrow down to your kernel but don't press enter just yet.  Press `e` instead to edit the boot parameters.  Arrow down to the bottom line and to the far right of that line.  Insert the word `printme` there (with a space after whatever came before).  Continue booting by pressing `Control-X`.  Note that you'll normally be able to simply press enter to boot your kernel, since we won't do much with boot parameters from here on out.

### Working outside the Virtual Machine

If you are passing the `kernel` parameter from outside the virtual machine, you can do so now, and you can set the `printme` parameter by adding it inside the quotes of the `append` parameter you pass to `qemu` or `kvm`.

### Step 3 - Verifying the Results

When the virtual machine boots, log in and run the following command to see if your log message was printed:

`dmesg | grep -i "Hello World from Me"`

Then, boot normally without adding this `printme` parameter (by hitting enter in GRUB when you select your kernel), and re-run this `dmesg | grep -i "Hello World from Me"`.  The message should only appear when you include the `printme` parameter!

## Create a diff Patch File for Submission

The kernel is a large source tree.  As a result, it is helpful to create a single file that captures the revisions you've made.  You can create a single file, known as a patch, that captures these differences (and allows others to automatically apply your changes to their own files automatically!).

### Working within Your Virtual Machine

The `pristine_linux` directory was set up to allow you to compare your work against the original kernel source tree.  You can create a patch by running the following from your user home directory:

```
cd linux-2.6.22.19
make clean
cd ..
cd pristine_linux
make clean
cd ..
diff -ruB linux-2.6.22.19 pristine_linux >my.patch
```

The `make clean` commands are important so that you don't diff the binary object files you built earlier when you create your patch.  Therefore, I suggest creating this only when you are ready to submit your work!

#### Copying Files from within Your Virtual Machine

If you are running Linux (or can otherwise install `libguestfs-tools`), you can run the following command to install a tool that can read your qcow2 disk image from your local computer, allowing you to copy files from your virtual machine!  You can install it and give it permissions to read your kernel modules with the following commands (run from your local machine):

```
sudo apt install libguestfs-tools
sudo chmod 644 /boot/vmlinuz-*
```

To mount the image, you can run the following command from the directory where your qcow2 image is located (that is, the directory from which you run your `qemu` command to boot the virtual machine):

```
mkdir mnt
guestmount -r -a local.qcow2 -m /dev/sda1 mnt
```

This mounts the image as read only (so you can copy from it, but not to it) with the `-r` flag (this is optional), and tells guestmount to mount the first partition `/dev/sda1` to your `mnt` directory on your local system.  Feel free to `cd` into `mnt` and copy files back to your local home directory!  For example, you might do this to copy your patch file into the home directory of the computer where you ran `qemu` from the virtual machine image:

```
cd mnt/home/user
cp my.patch ~
```

Then, suppose you are logged into a remote server like `stratus.ursinus.edu`, you could copy this to your local computer by opening a new terminal for your local machine and executing the following to copy the file to your local desktop for submission:

```
scp <your user name>@stratus.ursinus.edu:my.patch ~/Desktop
```

To unmount the image when you are done (this is important!), run

```
cd ~
umount mnt
```

### Working Outside Your Virtual Machine: Using git

One advantage of using git is that you can automatically create a patch file from your commit history.

Commit and push your changes to git as usual with the following commands:

```
git add init/main.c
git commit -m "Adding a printme boot parameter that prints to the screen"
git pull
git push
```

You can see a list of your commits by typing `git log`.  Each log entry will have a hexadecimal hash value that identifies the commit.  You can create a diff patch between two such commits by copying the hashes (I'll call them `firstcommit` and `lastcommit` here), and executing:

`git diff firstcommit lastcommit > my.patch`

Avoid creating a large diff patch by choosing log entries close to one another and more recent than the initial commits.  Choose commits that represent the work that you did on this project (the *real* commits that you made).

<!-- 
References:
Grub boot no graphical splash: https://superuser.com/questions/986875/grub2-is-it-possible-to-disable-graphics-drivers-through-kernel-boot-command
Download kernel: https://launchpad.net/linux/2.6.22/2.6.22.19
Download qemu: https://www.qemu.org/download/
Download QEMU for Windows: https://qemu.weilnetz.de/w64/
Linux Cross Reference: https://elixir.bootlin.com/linux/v2.6.22.19/source
apt-key update: https://www.linuxquestions.org/questions/linux-software-2/unable-to-connect-to-keys-gnupg-net-838576/
update certificates: https://serverfault.com/questions/891734/debian-wheezy-outdated-root-certificates
debugging: https://nickdesaulniers.github.io/blog/2018/10/24/booting-a-custom-linux-kernel-in-qemu-and-debugging-it-with-gdb/
-->
