+++
title = 'Setting Up WSL 2 and Installing Docker Engine on Windows ❄'
date = 2024-03-12T11:50:56+05:30
draft = false
+++
# Setting Up WSL 2 and Installing Docker Engine on Windows ❄
In this post, we'll explore how to enable WSL 2 in Windows machine. Once WSL 2 enabled, we
will check how we can install docker engine and run a container using that. 
<!--more-->

Windows Subsystem for Linux (WSL) has revolutionized the way developers work on Windows machines by bringing native Linux capabilities directly into the Windows environment. Combined with Docker Engine, it provides a powerful platform for developing and running containerized applications seamlessly. In this guide, we'll walk through the steps to set up WSL 2 and install Docker Engine on your Windows system.

## Containerization Concept 
Containerization is OS-based virtualization that creates multiple virtual units in the userspace, known as Containers. Containers share the same host kernel but are isolated from each other through private namespaces and resource control mechanisms at the OS level.

Refer: https://www.docker.com/resources/what-container/ to understand container basics.

## Docker Engine
Docker Engine is an open source containerization technology for building and containerizing your applications.

Refer: https://docs.docker.com/engine/ to understand docker engine basics


## Prerequisites
Before we begin, ensure that you have the following prerequisites:

1. Windows 10 or later (with Windows Update enabled)
2. Windows Subsystem for Linux 2 (WSL 2) enabled
3. An active internet connection

## Step 1: Enable WSL 2 on Windows

1. Open PowerShell as an Administrator.
2. Run the following command to enable WSL 2:
   > wsl --set-default-version 2
3. Restart your computer to apply the changes.
4. Check status using command
   > wsl --status

**Tip**: We can use new Terminal app from Windows instead of PowerShell: https://learn.microsoft.com/en-us/windows/terminal/install
Windows 11 already comes with it enabled. For Windows 10, we may need to install it manually. 

In case you do not see wsl itslef in your windows machine, 
follow, https://learn.microsoft.com/en-us/windows/wsl/install for installations.

## Step 2: Install a Linux Distribution

Preferred Linux distribution can be installed either from command line way or from Microsoft Store

Command line way

![Alt text](/pinchofcode/images/wsl.png)

Microsoft Store

![Alt text](/pinchofcode/images/ms_store.PNG)

Launch the installed Linux distribution to initialize the setup process.

Create a user account and password when prompted.

Once installed, here after when you type below commands

> wsl shutdown (This command will shutdown the WSL subsystem)

> wsl (This command will launch the WSL subsystem and installed subsystem Linux distributions)

![Alt text](/pinchofcode/images/ubuntu.png)

Update the package list and install necessary dependencies:
   ```
   > sudo apt update
   > sudo apt upgrade
   > sudo apt install -y git openssh-client
```
## Step 3:  Install Docker Engine in WSL 2

1. **Verify system requirements:**
   
   Refer: https://docs.docker.com/desktop/install/windows-install/
   One additional step; in older Windows Versions we may need to update the linux kernel update pacakage. In updated Windows versions we do not have to do this step.

   https://learn.microsoft.com/ro-ro/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package

2. **Install docker engine to WSL distribution**

    Refer: https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script
    We use a convenience script here
    
   > curl -fsSL https://get.docker.com -o get-docker.sh
   > chmod +x get-docker.sh
   > sudo ./get-docker.sh

3. **Post Installation Scripts**

   Refer: https://docs.docker.com/engine/install/linux-postinstall/

   Here we create a docker group and add our user to that group

    a. Create the docker group.
    > sudo groupadd docker

    b. Add your user to the docker group.
    > sudo usermod -aG docker $USER
    
    b. Log out and log back in so that your group membership is re-evaluated. 
    You can also run the following command to activate the changes to groups:
    > newgrp docker

    Many modern Linux distributions use systemd to manage which services start when the system boots. On Debian and Ubuntu, the Docker service starts on boot by default. To automatically start Docker and containerd on boot for other Linux distributions using systemd, run the following command:

    > sudo systemctl enable docker.service

4. **Time to Verify Installations**

Verify that you can run docker commands without sudo.
   > docker run hello-world

This will start running a container. We can try running a nginx container as well.

   > docker pull nginx

   > docker run --name mynginx -p 80:80 -d nginx

![Alt text](/pinchofcode/images/docker1.PNG)
![Alt text](/pinchofcode/images/nginx.PNG)

5. **If you would like to install docker compose plugin**

Refer: https://docs.docker.com/compose/install/linux/

    > sudo apt update
    > sudo apt install docker-compose-plugin
    > docker compose version

You can now leverage the power of Docker containers for developing and deploying applications seamlessly in your Windows environment.