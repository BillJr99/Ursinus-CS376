---
layout: default
permalink: /UbuntuDocker
title: "CS376: Operating Systems - Launching an Ubuntu virtual machine with Docker"

---

# Getting a Linux Development Environment

Several assignments need standard Linux development tools (`gcc`, `gdb`, `valgrind`). How you get them depends on your computer. **Find your situation in the table below and follow that path** — you do not need to read the sections that do not apply to you.

| Your situation                                             | What to do                                               |
| ---------------------------------------------------------- | -------------------------------------------------------- |
| You already run **Linux** (native or WSL2 on Windows)       | [Native Linux — No VM Needed](#native-linux--no-vm-needed) |
| You run **Windows** and want an isolated Linux environment  | [Docker on Windows](#docker-on-windows)                  |
| You run **macOS** (Intel or Apple Silicon)                  | [Docker on macOS](#docker-on-macos)                      |
| You run **Linux** but want a disposable container           | [Docker on Linux](#docker-on-linux)                      |

> These paths get you the *user-space* tools (`gcc`, `gdb`, `valgrind`) for the C assignments. The kernel projects use a separate QEMU virtual machine; see [Booting a Custom Linux Kernel](Projects/BootingCustomKernel) for that.

## Native Linux — No VM Needed

If you are already on a Linux machine (including **Windows with WSL2**, or a Linux desktop/laptop, or a lab machine), you do **not** need Docker or any virtual machine at all. Just install the tools directly:

```
sudo apt update
sudo apt install -y build-essential gdb valgrind git vim
```

To enable core dumps in your current directory (used in the [GDB and Valgrind](GDBValgrind) walkthrough):

```
ulimit -c unlimited
sudo sysctl -w kernel.core_pattern=core.%u.%p.%t
```

On WSL2 specifically, core dumps sometimes land under `/mnt/wslg/dumps`. That is all you need — skip the rest of this page and start coding. The Docker sections below are only for people who cannot or prefer not to install these tools directly.

# Using Docker for an Isolated Ubuntu Environment

If standard development tools like `gdb` and `valgrind` are not available on your computer, or you prefer an isolated, disposable environment, you can create a container from which to access them. The setup is the same on every platform except for a couple of OS-specific details called out below.  

## Installing Prerequisite Software

Install Docker and Git for **your** operating system using the matching section below, then continue to [Creating an SSH Key](#creating-an-ssh-key-to-log-into-your-virtual-machine), which is the same for everyone.

### Docker on Windows

* Install **Docker Desktop**: [Windows installer](https://docs.docker.com/desktop/install/windows-install/).
    * If you have the Windows Subsystem for Linux (WSL) installed, select the option during installation to use WSL. If you have VirtualBox or another virtualization tool installed (or do not have WSL), choose Hyper-V.
    * (If you already use WSL2, consider the [Native Linux](#native-linux--no-vm-needed) path instead — it is simpler.)
* Install **Git**: [Git for Windows](https://git-scm.com/downloads).
* When you reach [Launching the Image](#launching-the-image), use the **Windows** volume-mount example (`/c/Users/<your user name>/...`).

### Docker on macOS

* Install **Docker Desktop**: [Mac installer](https://docs.docker.com/desktop/install/mac-install/).
    * The download page asks whether you want the Intel (x86) or Apple Silicon version. Click the Apple menu > "About This Mac": if it says Intel, choose x86/x64; if it says M1/M2/M3 or "Apple", choose Apple Silicon.
* Install **Git**: [Git for Mac](https://git-scm.com/download/mac), or via a package manager:
    * `sudo port install git` if you have [MacPorts](https://www.macports.org/), or `brew install git` if you have [Homebrew](https://brew.sh/).
    * If you have neither (or the command fails), run `sudo xcode-select --install`, then install [MacPorts](https://www.macports.org/) or [Homebrew](https://brew.sh/) and re-run the appropriate command.
* When you reach [Launching the Image](#launching-the-image), use the **macOS** volume-mount example (`/Users/<your user name>/...`).

### Docker on Linux

* Install **Docker Engine** from your distribution's packages or [Docker's Linux install guide](https://docs.docker.com/engine/install/), then add yourself to the `docker` group (`sudo usermod -aG docker $USER`) and log out/in.
* Install **Git**: `sudo apt install git` (or your distribution's equivalent).
* When you reach [Launching the Image](#launching-the-image), use the **Linux** volume-mount example (`/home/<your user name>/...`).
* (Most Linux users are better served by the [Native Linux](#native-linux--no-vm-needed) path — Docker here is only for an isolated/disposable container.)
    
## Creating an SSH Key to Log Into Your Virtual Machine

Next, open a terminal and run the following command, if you don't already have an SSH public key on your system.

```
ssh-keygen -t ed25519
```

Follow the prompts and it will create a public key file within your home directory called `~/.ssh/id_rsa.pub` (as well as another file in that same directory called `id_rsa`, which is your associated private key; don't share that file with anyone!).  You will use the contents of this file later; to copy these contents, type the following command, and copy the output to your clipboard:

```
cat ~/.ssh/id_rsa.pub
```

## Using Docker

Launch the Docker Desktop tool.  You will be prompted to accept the license agreement and log in (and perhaps create a Docker account) the first time you launch it.

Now, launch the terminal, where we will create our virtual image.  Run this command to gain access to the ubuntu base image (which we will customize):

```
docker pull ubuntu
```

### Creating a Dockerfile

Create a file named `Dockerfile` with the following contents, **making sure to set the `ENV` variables at the beginning of the file with your host SSH public key `id_rsa.pub` file contents, and a password for the default account on the guest**:

```
# Use the official image as a parent image
FROM ubuntu

# Specify the SSH public key as a string variable (replace with your public key)
ENV SSH_PUBLIC_KEY="<paste the contents of your id_rsa.pub file here>"

# Specify the password for user and root user as a string variable (replace with your desired password)
ENV USER_PASSWORD="<put a good password that you'll remember here>"

# Update the system, install OpenSSH Server, and set up users
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y openssh-server openssh-client aptitude sudo && \
    apt-get install -y git build-essential libc6-dbg gdb valgrind vim && \
    apt-get install -y inetutils-ping traceroute python3 python3-pip

# Create user and set password for user and root user
RUN useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 ubuntu && \
    echo "ubuntu:$USER_PASSWORD" | chpasswd && \
    echo "root:$USER_PASSWORD" | chpasswd

# Set up the ubuntu user's environment to generate core files in the current directory
RUN echo "ulimit -c unlimited" >> /home/ubuntu/.bashrc
RUN sysctl -w kernel.core_pattern=core.%u.%p.%t

# Set up configuration for SSH
RUN mkdir -p /var/run/sshd && \
    sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
    echo "export VISIBLE=now" >> /etc/profile

# Copy the SSH public key to the authorized_keys file of the ubuntu user
RUN mkdir -p /home/ubuntu/.ssh && \
    echo "$SSH_PUBLIC_KEY" >> /home/ubuntu/.ssh/authorized_keys && \
    chown -R ubuntu:root /home/ubuntu/.ssh && \
    chmod 700 /home/ubuntu/.ssh && \
    chmod 600 /home/ubuntu/.ssh/authorized_keys

# Generate an SSH key pair during image build for the ubuntu user
RUN ssh-keygen -t rsa -b 2048 -f /home/ubuntu/.ssh/id_rsa -N "" && \
    chown -R ubuntu:root /home/ubuntu/.ssh

# Create a volume at the SHARE_PATH
VOLUME ["/home/ubuntu/shared"]

# Expose the SSH port
EXPOSE 22

# Run SSH
CMD ["/usr/sbin/sshd", "-D"]
```

### Building the Image

Build this custom image with the following command:

```
docker build -t ubuntu-ssh . 
```

## Launching the Image

To start the image, run the `docker run` command **for your operating system** (they differ only in the host path before the `:` in the `-v` volume mount):

**Windows** (paths under `C:\Users`):

```
docker run -d -v /c/Users/<your user name>/Desktop:/home/ubuntu/shared/desktop -p 2222:22 --privileged ubuntu-ssh
```

**macOS** (paths under `/Users`):

```
docker run -d -v /Users/<your user name>/Desktop:/home/ubuntu/shared/desktop -p 2222:22 --privileged ubuntu-ssh
```

**Linux** (paths under `/home`):

```
docker run -d -v /home/<your user name>/Desktop:/home/ubuntu/shared/desktop -p 2222:22 --privileged ubuntu-ssh
```

The `-v` path including `<your user name>` shares a local directory inside your container's home directory. This example mounts your local `Desktop` at `/home/ubuntu/shared/desktop`, but you can choose another directory, or omit the entire `-v` parameter to skip sharing altogether.

### Logging into the Image

You can log into the image using `ssh`:

```
ssh ubuntu@localhost -p 2222 
```

A password is not required, since we provided our ssh key.

### Enabling Access to GitHub

[Log into GitHub, and copy](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) both the contents of your local computer `id_rsa.pub` and the contents of your guest `id_rsa.pub` to the SSH Keys section of GitHub's settings.  To list the contents of your virtual image public key file, type the following command while logged into the virtual machine (just like you did earlier on the host), and copy the resulting output:

```
cat ~/.ssh/id_rsa.pub
```

# Quickstart: Launching a Vanilla Ubuntu Image

You can quickly spin up an Ubuntu instance by typing:

```
docker pull ubuntu
docker run -it ubuntu
```

However, this will not have incoming SSH access or any of the software packages we use.  You'll have to install those manually.

# References

* [https://tecadmin.net/setting-up-ubuntu-docker-container-with-ssh-access/](https://tecadmin.net/setting-up-ubuntu-docker-container-with-ssh-access/)
* [https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host)
