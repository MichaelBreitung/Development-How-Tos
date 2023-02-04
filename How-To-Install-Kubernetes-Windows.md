# How to Install Kubernetes for Windows

The setup below will allow you to run a local *Kubernetes* cluster inside a virtual machine on Windows and control it from within a WSL Linux environment running on the same Windows machine. A prerequisite is a WSL environment, which can be setup using [this guide](https://github.com/MichaelBreitung/Development-How-Tos/blob/master/How-To-Setup-WSL-Dev-Env.md).

## Minikube

Minikube is used to provide a local Kubernetes cluster.

### Installation

1) Install the latest version of Virtual Box - VBox -  from [here](https://www.virtualbox.org/wiki/Downloads).

   - If you get an error that *Python Core / win32api* are missing:

     - Install [Python for Windows](https://www.python.org/downloads/windows/)

     - Install *win32api*: 

       ````
       python -m pip install pywin32
       ````

2) Use the Windows Package Manager to install *Minikube*. In *CMD* or *Powershell*:

   ````
   winget install minikube
   ````

3) Start *Minikube*

   ````
   minikube start --driver=virtualbox --no-vtx-check
   ````

   This command will create a virtual machine called *minikube*. It will setup a *Host-Only* network adapter, which lets you access the Kubernetes cluster from Windows and WSL.

   We don't check for *vtx* because it's not required to run the Kubernetes cluster in Virtual Box. Activating it can interfere with WSL.

4. Check the status of the cluster:

   ````
   minikube status
   ````

5. During the start process, a few files will be created, which are required inside WSL. Copy the following files into a folder that is shared with WSL. Sharing folders is shown in [this guide](https://github.com/MichaelBreitung/Development-How-Tos/blob/master/How-To-Setup-WSL-Dev-Env.md).

   ````
   C:\Users\< your user name >\.minikube\profiles\minikube\config
   C:\Users\< your user name >\.minikube\ca.crt
   C:\Users\< your user name >\.minikube\profiles\minikube\client.crt
   C:\Users\< your user name >\.minikube\profiles\minikube\client.key
   ````

### Good to Know

- Once you are done with you local Kubernetes cluster, you can stop it:

  ````
  minikube stop
  ````

- To start it again, you don't need to use the options from point 3 again. Just use:

  ````
  minikube start
  ````

- To get the IP address of the Kubernetes cluser, use:

  ````
  minikube ip
  ````

- SSH into minikube VM for debugging:

  ````
  minikube ssh
  ````

- SSH into minikube from WSL:

  ````
  ssh docker@< ip of minikube cluster >
  
  # password is
  tcuser
  ````

## Kubectl

This tool is used to control the cluster. 

### Installation

1) Install it inside your WSL Linux distro:

   ````
   # get the latest kubeectl release
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   
   # download the kubectl checksum file
   curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
   
   # validate the binary against the checksum -> kubectl: OK
   echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
   
   # install kubectl
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   
   # check kubectl version
   kubectl version --client --output=yaml
   ````

2. Use the *Minikube* configuration:

   ````
   # go to your users home
   cd ~
   
   # if no .kube folder exists, create it
   mkdir .kube
   
   # copy configuration files from shared folder - see 5. of Minikube installation
   cd .kube
   cp -r < path to shared folder >/* .
   ````

3. Edit the copied *config* file using VS Code or an editor of your choice. You have to adjust the paths to the different certification and key files:

   ````
   apiVersion: v1
   clusters:
   - cluster:
       certificate-authority: ca.crt
   ...
   ...
   users:
   - name: minikube
     user:
       client-certificate: profiles/minikube/client.crt
       client-key: profiles/minikube/client.key
   ````

4. Check if the cluster can be accessed:

   ````
   kubectl cluster-info
   ````

   This command will use the edited *config* inside the *~/.kube* folder to create a connection to the cluster which is running in a virtual machine outside of WSL. The result of the call should read something like this:

   ````
   Kubernetes control plane is running at https://< ip of Host Network if of VM >:< port of VM >
   CoreDNS is running at https://< ip of Host Network if of VM >:< port of VM >/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
   
   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   ````


At this point, everything is ready to use the cluster from WSL. This is a good setup for learning how to use Kubernetes and for testing configurations.

