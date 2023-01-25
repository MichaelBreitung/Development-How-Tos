# How To Set up a Linux Development Environment Under Windows

When working with tools like Docker and Kubernetes, Linux provides more flexibility than Windows if the focus is solely on Linux containers. That's why, even on a Windows system, it makes sense to install a Linux Environment for development.

With the **W**indows **S**ubsystem for **L**inux Version 2, Windows 10 and 11 provide a great option to get access to a Linux environment. There's no need to install and setup a Virtual Machine using tools like Hyper-V or Virtual Box. Available for WSL 2 are several Linux distributions, including Ubuntu. There's also a plugin for Visual Studio Code that provides interoperability between Windows and WSL for development. 

This guide focuses on the installation of Ubuntu via WSL, Docker, and the setup of Visual Studio Code as development environment.

## WSL 2 Ubuntu

### Installation

This part focusses on the installation of WSL 2 with the Ubuntu distribution from the *Microsoft Store*. Included is the installation of important packages, the setup of Remote Desktop access as well as troubleshooting along the way.

1. In *Windows Search* type "Turn Windows Features on or off" and find the *Windows Subsystem for Linux* setting. Make sure it is activated. Also, check that *Virtual Machine Platform* is active. 

2. Open the *Microsoft Store* and make sure that *Windows Subsystem for Linux* is installed. In the *cmd* window you can check its version and status:

   ````
   # display version -> Note that this is the installed version. Even if your installed WSL version supports WSL 2, 
   # the returned version will be something like 1.0.3.0 or higher
   wsl --version
   
   # set default version to WSL 2 -> Note this does not change the installed version, 
   # just the default version used to run distributions like Ubuntu. If this command is successful, 
   # your installed WSL can use Version 2 features
   wsl --set-default-version 2
   
   # display status -> This will show you the selected default WSL version, which should be 2
   wsl --status
   ````

3. Install the Ubuntu distro from the *Microsoft Store*. Just search for Ubuntu and install the App.

4. Confirm that Ubuntu is using the right version of WSL:

   ````
   # This should show VERSION 2
   wsl -l -v
   
   # If it's still VERSION 1 use
   wsl --set-version <Ubuntu Distro Name> 2
   ````

5. Start Ubuntu - you'll find it in the *Start* menu.

   - You will be asked to create a user, providing a name and password.

6. Enable the copy and paste feature:

   1) Right-Click on the Ubuntu window
   2) Go to *Properties*
   3) Under *Edit Options* activate *Use Ctrl+Shift+C/V as Copy/Paste*

7. If you are using special DNS servers, setup the DNS Servers inside Ubuntu. If not, continue with step 8.

   1) Switch to interactive mode, which lets you act as root user:

      ````
      sudo -i
      ````

   2. Add the following two lines to */etc/wsl.conf*:

      ````
      [network]
      generateResolvConf = false
      ````

   3. Restart WSL:

      ````
      # in windows cmd or Powershell
      wsl --shutdown
      
      # use wsl --status to get the correct name of your Ubuntu. Then:
      wsl -d <Ubuntu distro>
      ````

   4. Get the DNS configuration of the System. 

      ````
      # in cmd or Powershell under Windows
      ipconfig /all
      ````
      You'll find two lines under the entry *DNS Servers*, if custom servers have been set up.

   5. Open Ubuntu again by clicking on App Icon.

   6. Inside Ubuntu, create a file *resolv.conf* and add the following two lines:

      ````
      nameserver <first DNS Servers IP>
      nameserver <second DNS Servers IP>
      ````

   7. Restart WSL.

8. Install required packages:

   ````
   sudo -i
   
   # update the package list and upgrade the installed packages to the newest versions
   apt update && apt upgrade -y
   
   # install additional packages - those will allow you to get a graphical user experience via Remote Desktop
   apt install ubuntu-desktop xfce4 dbus-x11 xrdp net-tools -y
   
   # install python packages
   apt install python3 python3-pip
   
   # remove packages that are not required - sometimes additional packages get installed during apt install
   apt autoremove -y
   ````

   - If prompted to select a display manager, use the default which is *gdm3*.
   - If there are any errors that servers cannot be reached, this means that the DNS setup is not correct. Refer to step 7 to solve this problem or try the following as a temporal fix:

   ````
   echo "nameserver 8.8.8.8" | tee /etc/resolv.conf > /dev/null
   ````

9. To be able to connect from Remote Desktop using *localhost* instead of the ever changing IP of WSL, perform the following commands:

   ````
   # create a backup of xrdp.ini first
   cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
   
   # change some settings in xrdp.ini
   sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini
   sed -i 's/max_bpp=32/#max_bpp=32\nmax_bpp=128/g' /etc/xrdp/xrdp.ini
   sed -i 's/xserverbpp=24/#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini
   ````

10. Start xrdp:
      ````
      /etc/init.d/xrdp start

      # check if it's running
      /etc/init.d/xrdp status
      ````

11. Add this command to */etc/rc.local* so xrdp starts automatically, once you start Ubuntu:

    ````
    sudo /etc/init.d/xrdp start
    exit 0
    ````

WSL Ubuntu is now ready to be used. You can use Remote Desktop under Windows to connect to it via *localhost:3390*. This allows you to interact with the GUI of *xfce4* we installed above. If you don't need an UI, just use the command line.

### Good to Know

- From inside WSL Ubuntu, you can access your Windows *c* drive via */mnt/c*. It is automatically mounted during startup. If you want to have a logical folder to share files between Ubuntu and Windows, you can create a *wsl-share* folder and create a symlink to it for quick access:

  ````
  # While in your home folder, perform the following command - I have a wsl-share folder under c:\\ under Windows
  ln -s /mnt/c/wsl-share wsl-share
  ````

### Troubleshooting

#### Remote Desktop and xrdp

If you get a black screen when connecting to WSL via Remote Desktop, ensure that you are not logged in with the same user anywhere else - the Ubuntu CLI, for example.

If this doesn't solve the problem, add the following lines inside */etc/xrdp/startwm.sh* before the line *if test -r /etc/profile; then...*:

````
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
. $HOME/.profile
````

If there is already a session running on the system blocking or holding resources that xrdp needs to create a new session, xrdp may not be able to initiate a new session and the connection may fail or result in a black screen.

By unsetting the *DBUS_SESSION_BUS_ADDRESS* and *XDG_RUNTIME_DIR* environment variables and sourcing the *.profile* file, we create a new session environment, which helps to avoid conflicts.

### References

-  https://www.bleepingcomputer.com/news/microsoft/windows-10-wsl-now-can-run-linux-commands-on-startup/
-  https://www.youtube.com/watch?v=OlBmvOKvUkQ
-  https://youtu.be/EAxRNoRRvFQ

## Docker

### Installation

1. Inside Ubuntu, install Docker, by following [this guide](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

2. Add yourself to the *docker* group to be able to perform docker commands without *sudo*:

   ````
   # add your user to docker group
   sudo usermod -aG docker < user name >
   
   # logout and in of user, to have group changes become effective
   su - < user name >
   ````
3. Setup a secure credentials store, which will be used when you login to [Docker Hub](https://hub.docker.com/):
   - Simple setup using *DBus Secret Service* -> https://luiscachog.io/docker-login-the-right-way/
   - More complex setup using *pass* -> https://brain2life.hashnode.dev/how-to-set-up-secure-local-credential-storage-for-docker-on-ubuntu-2004

## Visual Studio Code

The intention of installing the Ubuntu Distribution via WSL was to use it for development. We can use Python and Git there and connect VS Code to Ubuntu. This way, you can still develop your code and edit files in VS Code under Windows, while you use the tooling inside Ubuntu. 

### Installation

1. Get the installer from [here](https://code.visualstudio.com/) and install VS Code.

2. Install the following extension in VS Code:

   - WSL

3. Inside Ubuntu, create a *Develop* folder inside your home. 

   - Inside this folder, you can create and / or checkout any project folders you want to work on. As an alternative, you can also head to */mnt/c* and create the *Develop* folder there. Just create a symbolic link to this folder from your home and use this folder for all your development. This gives you the most flexibility because you can directly access it from Windows.

4. Inside one of your project folders within *Develop* type:

   ````
   code .
   ````

   The first time you do this, extensions are installed and the linkage to VS Code is created. VS Code then starts as usual. But it will be the WSL version. 

5. Head to the extensions inside VS Code.

6. Install the following extensions for Ubuntu:

   - Python - for proper VS Code Python integration
   - Mypy - for Python type checking
   - Docker - helps managing docker files, images and containers

Now you are fully setup for Python development under either Linux, Windows or both. You also have access to Docker from inside VS Code command line.

###	References

- https://code.visualstudio.com/docs/remote/wsl
- https://code.visualstudio.com/docs/remote/wsl-tutorial
