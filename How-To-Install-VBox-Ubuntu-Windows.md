# How to Install a Virtual Box with Ubuntu

The most convenient Linux development environment under Windows can be [set up via WSL as shown here](https://github.com/MichaelBreitung/Development-How-Tos/blob/master/How-To-Setup-WSL-Dev-Env.md). But for proper testing, you might sometimes need additional machines. For this use case, you can set up one or several virtual machines using Virtual Box.

## Installation

1) Install the latest version of Virtual Box - VBox -  from [here](https://www.virtualbox.org/wiki/Downloads).

   - If you get an error that *Python Core / win32api* are missing:

     - Install [Python for Windows](https://www.python.org/downloads/windows/)

     - Install *win32api*: 

       ````
       python -m pip install pywin32
       ````

2. Download one of the latest **Ubuntu** images from [here](https://ubuntu.com/download/desktop).

3. Start VBox and create a new Virtual Machine, using the following details:

   - Select the downloaded Ubuntu ISO image as source.
   - Skip the unattended installation, because it currently causes [problems](https://askubuntu.com/questions/1435918/terminal-not-opening-on-ubuntu-22-04-on-virtual-box-7-0-0).
   - Select a sufficient amount of RAM based on what's available on your system. **8GB** is a good amount.
   - Select at least **4 CPUs** if available.
   - For Ubuntu, a disk space of **25GB** should be sufficient. When specifying this size, think about what you plan to use this virtual machine for and what you are going to install.

4. Right-click on the machine inside VBox and go to the *Settings->Display* and set the *Video Memory* to **128 MB**.

5. To start the installation, start the virtual machine and follow the instructions. A minimal installation should be sufficient for most development use cases. You should rather install the necessary packages later.

6. Open *Terminal* and install build essentials and net-tools:

   ````
   sudo apt update
   sudo apt upgrade -y
   sudo apt install -y build-essential linux-headers-$(uname -r)
   sudo apt install -y net-tools
   ````

7. For an attended installation, *VBox Guest Additions* are not automatically installed. To install it:

   1. Click on *Devices->Insert Guest Additions CD Image...* in the menu bar of Virtual Box.
   2. The mounted image will appear on the left inside the Ubuntu UI. 
   3. Inside *Terminal* head to *media/< your user name >/VBox_GAs_...*
   4. Execute *autorun.sh*.

8. Once the installation is finished, restart the Virtual Box.

9. Activate *copy and paste* via *Settings->General->Advanced->Shared Clipboard (Bidirectional)*.

Your Ubuntu desktop should now resize to the size of the parent window, and you should be able to copy and paste between Windows and Ubuntu. Now it's time to install any additional packages you require, for example Docker following [this guide](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

## Network Setup

Per default, Virtual Box uses the so called *NAT* network setting. This allows a Guest OS to connect to the Internet via the host system's network adapter. The Guest OS can also access WSL that way. What the *NAT* setting does not allow you to do is accessing the Guest OS from the Host or LAN, including WSL. To allow such access, setup port forwarding for the required applications.

As an alternative, you can switch to *Host-only Adapter* in the Network settings of VBox. With this setting, you can access the VBox Guest OS from the host and from WSL. But you will no longer be able to access WSL from the Guest OS.

If you require a bidirectional communication between a VBox Guest OS and WSL, you can install two adapters inside the Guest OS: *NAT* and *Host-only Adapter*. You can [use *netplan* for this](https://askubuntu.com/questions/984445/netplan-configuration-on-ubuntu-17-04-virtual-machine).

