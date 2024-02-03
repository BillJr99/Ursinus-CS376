---
layout: default
permalink: /UbuntuDocker
title: "CS376: Operating Systems - Launching an Ubuntu virtual machine with Docker"
excerpt: "CS376: Operating Systems - Launching an Ubuntu virtual machine with Docker"

---

# Launching an Ubuntu virtual machine with Docker

If standard development tools like `gdb` and `valgrind` are not available on your computer, you can create a virtual machine from which to access them.  

## Installing Prerequisite Software

First, install the following tools on your computer:

1. Docker
    * [Windows](https://docs.docker.com/desktop/install/windows-install/)
        * If you have the Windows Subsystem for Linux (WSL) installed, select the option during the installation to use WSL.  If you have VirtualBox or another virtualization software installed (or otherwise do not have WSL installed), choose Hyper-V.
    * [Mac](https://docs.docker.com/desktop/install/mac-install/)
        * The Mac download page will ask if you want to install the x86 or Apple Silicon version of the software.  If you are running an ARM processor, choose the Apple Silicon version.  If you aren't sure, click the Apple menu at the top left, and choose "About this Mac."  If it says Intel, it's an x86/x64 installation, and if it says M1 or M2, it's an ARM installation.
      
2. Git
    * [Windows](https://git-scm.com/downloads)
    * [Mac](https://git-scm.com/download/mac)
        * Open a terminal and execute `sudo port install git` if you have [MacPorts](https://www.macports.org/) installed, or `brew install git` if you have [Homebrew](https://brew.sh/) installed.  
        *  If you don't have either MacPorts or Homebrew installed (or if the command above fails), run the command `sudo xcode-select –install` in your terminal, and then download and install either [MacPorts](https://www.macports.org/) or [Homebrew](https://brew.sh/), and then re-run the appropriate install command above.
    
## Creating an SSH Key to Log Into Your Virtual Machine

Next, open a terminal and run the following command, if you don't already have an SSH public key on your system.

```
ssh-keygen -t rsa
```

Follow the prompts and it will create a public key file within your home directory called `~/.ssh/id_rsa.pub` (as well as another file in that same directory called `id_rsa`, which is your associated private key; don't share that file with anyone!).

## Using Docker

Launch the Docker Desktop tool.  You will be prompted to accept the license agreement and log in (and perhaps create a Docker account) the first time you launch it.

Now, launch the terminal, where we will create our virtual image.  Run this command to gain access to the ubuntu base image (which we will customize):

```
docker pull ubuntu
```

### Creating a Dockerfile

Create a file named `Dockerfile` with the following contents, making sure to set the `ENV` variables at the beginning of the file:

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
    apt-get install -y git build-essential libc6-dbg gdb valgrind vim

# Create user and set password for user and root user
RUN useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 ubuntu && \
    echo "ubuntu:$USER_PASSWORD" | chpasswd && \
    echo "root:$USER_PASSWORD" | chpasswd

# Set up configuration for SSH
RUN mkdir /var/run/sshd && \
    sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
    echo "export VISIBLE=now" >> /etc/profile

# Copy the SSH public key to the authorized_keys file of the ubuntu user
RUN mkdir /home/ubuntu/.ssh && \
    echo "$SSH_PUBLIC_KEY" >> /home/ubuntu/.ssh/authorized_keys && \
    chown -R ubuntu:root /home/ubuntu/.ssh && \
    chmod 700 /home/ubuntu/.ssh && \
    chmod 600 /home/ubuntu/.ssh/authorized_keys

# Create a directory for the SSH key and set permissions
RUN mkdir /home/ubuntu/.ssh && chmod 700 /home/ubuntu/.ssh

# Generate an SSH key pair during image build for the ubuntu user
RUN ssh-keygen -t rsa -b 2048 -f /home/ubuntu/.ssh/id_rsa -N "" && \
    chown -R ubuntu:ubuntu /home/ubuntu/.ssh

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

To start the image, run the following command (which you can skip to once you've built the image above):

```
docker run -d -v /c/Users/<your user name>/Desktop:/home/ubuntu/shared/desktop -p 2222:22 ubuntu-ssh 
```

The path including `<your user name>` allows you to share a local directory within your virtual image home directory!  This example will mount your local `Desktop` directory at `/home/ubuntu/shared/desktop`, but you can select other possibilities here.  Feel free to specify this or omit the entire `-v` parameter to skip it.

### Logging into the Image

You can log into the image using `ssh`:

```
ssh ubuntu@localhost -p 2222 
```

A password is not required, since we provided our ssh key.

### Enabling Access to GitHub

Log into GitHub, and copy both your local computer `id_rsa.pub` and the contents of your guest `id_rsa.pub` to the SSH Keys section of GitHub's settings.  To list the contents of your virtual image public key file, type the following command while logged into the virtual machine:

```
cat ~/.ssh/id_rsa.pub
```

## References

* [https://tecadmin.net/setting-up-ubuntu-docker-container-with-ssh-access/](https://tecadmin.net/setting-up-ubuntu-docker-container-with-ssh-access/)
* [https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host)