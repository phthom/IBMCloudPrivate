



<div style="background-color:black;color:white; vertical-align: middle; text-align:center;font-size:250%; padding:10px; margin-top:100px"><b>
 Installing IBM Cloud Private
 </b></a></div>

---
# Single Node Installation of IBM Cloud Private
---

![ICP Logo](./images/logoicp.png)



# Table of Content

- [1. IBM Cloud Private Overview](#1-ibm-cloud-private-overview)
    + [What is a private Cloud ?](#what-is-a-private-cloud--)
    + [Terminology](#terminology)
    + [Architecture](#architecture)
- [2. Objectives](#2-objectives)
- [3. Prerequisites](#3-prerequisites)
- [4. First Option: Install ICP on Ubuntu VM](#4-first-option--install-icp-on-ubuntu-vm)
    + [Task 1 : Configuring the system](#task-1---configuring-the-system)
    + [Task 2 : Add Docker’s official GPG key](#task-2---add-docker-s-official-gpg-key)
    + [Task 3: Add a repo to get the Docker](#task-3--add-a-repo-to-get-the-docker)
    + [Task 4: Get Docker](#task-4--get-docker)
    + [Task 5: Download IBM Cloud Private](#task-5--download-ibm-cloud-private)
    + [Task 6: SSH Keys setup](#task-6--ssh-keys-setup)
    + [Task 7: Customize ICP](#task-7--customize-icp)
    + [Task 7: Install ICP](#task-7--install-icp)
    + [Task 8: Install and configure CLIs  (kubectl, ibmcloud pr, ibmcloud dev)](#task-8--install-and-configure-clis---kubectl--ic-px--ic-dev-)
    + [Task 9: Adding persistent storage to Kubernetes](#task-9--adding-persistent-storage-to-kubernetes)
    + [Task 10: End of installation](#task-10--end-of-installation)
- [5. Second Option: Using Vagrant and VirtualBox](#5-second-option--using-vagrant-and-virtualbox)
- [6. Conclusion](#6-conclusion)
    + [Results](#results)
- [End of Lab](#end-of-lab)
- [appendix A : Preparing your VM (VMWare) with Ubuntu](#appendix-a---preparing-your-vm--vmware--with-ubuntu)
    + [Task A1 - Download Ubuntu](#task-a1---download-ubuntu)
    + [Task A2 - Create a VM](#task-a2---create-a-vm)
    + [Task A3 - Set the language](#task-a3---set-the-language)
    + [Task A4 - Set the keyboard](#task-a4---set-the-keyboard)
    + [Task A5 - Install Ubuntu Server](#task-a5---install-ubuntu-server)
    + [Task A6 - Select Location](#task-a6---select-location)
    + [Task A7 - Select locales](#task-a7---select-locales)
    + [Task A8 - Choose a hostname](#task-a8---choose-a-hostname)
    + [Task A9 - Choose a username](#task-a9---choose-a-username)
    + [Task A10 - Type a password](#task-a10---type-a-password)
    + [Task A12 - Partition your disk](#task-a12---partition-your-disk)
    + [Task A13 - No Proxy](#task-a13---no-proxy)
    + [Task A14 - Launch the installation](#task-a14---launch-the-installation)
    + [Task A15 - Add some packages](#task-a15---add-some-packages)
    + [Task A16 - GRUB](#task-a16---grub)
    + [Task A17 - Login for the first time](#task-a17---login-for-the-first-time)
    + [Task A18 - Get the IP address](#task-a18---get-the-ip-address)
    + [Task A19 - SSH to the VM](#task-a19---ssh-to-the-vm)
    + [Task A20  - Update the OS](#task-a20----update-the-os)
    + [Task A21 - Define the network interface](#task-a21---define-the-network-interface)
    + [Task A22 - You are now ready to start the ICP installation](#task-a22---you-are-now-ready-to-start-the-icp-installation)
- [appendix B : Changing ICP admin password](#appendix-b---changing-icp-admin-password)
    + [Task B1 - Login to your ICP cluster using ssh](#task-b1---login-to-your-icp-cluster-using-ssh)
    + [Task B2 - Generate your new ICP password in base64](#task-b2---generate-your-new-icp-password-in-base64)
    + [Task B3 - Edit ICP secrets](#task-b3---edit-icp-secrets)
    + [Task B4 - Delete the auth pods](#task-b4---delete-the-auth-pods)
    + [Task B5 - Test the new password](#task-b5---test-the-new-password)
- [End of Appendix](#end-of-appendix)



# 1. IBM Cloud Private Overview

**IBM Cloud Private** is a private cloud platform for developing and running workloads locally. It is an integrated environment that enables you to design, develop, deploy and manage on-premises, containerized cloud applications behind your firewall. It includes the container orchestrator Kubernetes, a private image repository, a management console and monitoring frameworks.

### What is a private Cloud ?

 Private cloud is a cloud-computing model run solely for one organization. It can be managed internally or by third party, and can be hosted behind the company firewall or externally. Private cloud offers the benefits of a public cloud, including rapid deployment and scalability plus ease of use and elasticity, but also provides greater control, increased performance, predictable costs, tighter security and flexible management options. Customize it to your unique needs and security requirements.

### Terminology

- **master node** :  
    - controls and manages the cluster
    - **Kubectl**:  command line client
    - REST API:  used for communicating with the workers
    - Scheduling and replication logic
    - Generally 3 or more master nodes for resiliency
- **worker node**
    - is a worker machine in Kubernetes, previously known as a minion. A node may be a **VM** or **physical machine**, depending on the cluster. Each node has the services necessary to run pods and is managed by the master components.
    - **Kubelet**  agent that accepts commands from the master
    - **Kubeproxy**: network proxy service on a node level, responsible for routing activities for inbound or ingress traffic
    - Docker host
- **Containers**: Units of packaging
- **Pods**:
    - A collection of containers that run on a worker node
    - A pod can contain more than one service
    - Each pod has it’s own IP
    - A pod shares a PID namespace, network, and hostname
- **Replication Controller** :
    - Ensures availability and scalability
    - Responsible for maintaining as many pods as requested by the user
    - Uses a template that describes specifically what each pod should contain
- **Labels**: 
    - Metadata assigned to K8 resources – such as pods, services
    - Key-Value pairs for identification
    - Critical to K8s as it relies on querying the cluster for resources that have certain labels
- **Services:**
    - Collection of pods exposed as an endpoint
    - Information stored in the K8 cluster state and networking info propagated to all worker nodes
- **Secrets**:
    - Sensitive information that containers need to read or consume
    - Are special volumes mounted automatically so that the containers can read its contents
    - Each entry has it’s own path


### Architecture

See below a global picture showing ICP architecture and most common components.

![Architecture](./images/architecture.png)

This lab is going to focus on the couple "Docker & Kubernetes" which is the main foundation of IBM Cloud Private.

# 2. Objectives


In this first lab, you will prepare and install IBM Cloud Private (ICP) on a **single node** (host) running on a VM (thru VMware or it could be a Virtual Server in a Cloud or a VirtualBox).

You will learn how to:

- prepare your platform to support a single node ICP (i.e all nodes - master, worker, proxy, boot nodes will be concentrated in one node)
- configure the installation setup for ICP
- check and validate all the prerequisites
- install Docker and Hyperkube
- check all the components


# 3. Prerequisites

This lab needs some hardware and software prerequisites.

At least (minimal):
- [ ] one host (physical or virtual)
- [ ] CPU = 8 cores (or virtual cores)
- [ ] RAM = 10 GB (10,240 MB)
- [ ] Storage = 40 GB (depending on the other solutions that you install on top of ICP)

Depending on the installation types:
- [ ] First option: Ubuntu 16.04 (Sever or Desktop) in a **VMware VM**
- [ ] Second Option:  Vagrant and **VirtualBox**

If you need some more help, see the official documentation: [here](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/kc_welcome_containers.html)

or

https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/kc_welcome_containers.html

# 4. First Option: Install ICP on Ubuntu VM

Be sure that you have already prepared a VMware VM (see Appendix A) or a virtual server containing a **Ubuntu** system up and running. Login to the Ubuntu system as **root** or sudo -i once connected.

> **ipaddress** is the ip address of the VM and subsequently the ip address of the different nodes in this lab.

Here are the steps:

### Task 1 : Configuring the system

Before updating the operating system, check the **/etc/hosts** file in Linux.

Use **putty** or **ssh** to get connected to the ubuntu VM.

`ssh root@ipaddress`

![login to the system](./images/ssh2.png)

Once connected, check your (public) ip address by typing the command:

`ifconfig`

Results:
```console
# ifconfig
eth0      Link encap:Ethernet  HWaddr 06:41:88:4e:6f:f3  
          inet addr:10.45.154.246  Bcast:10.45.154.255  Mask:255.255.255.192
          inet6 addr: fe80::441:88ff:fe4e:6ff3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1813 errors:0 dropped:0 overruns:0 frame:0
          TX packets:57 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:144586 (144.5 KB)  TX bytes:4102 (4.1 KB)

eth1      Link encap:Ethernet  HWaddr 06:29:fc:86:31:fa  
          inet addr:158.175.102.12  Bcast:158.175.102.15  Mask:255.255.255.240
          inet6 addr: fe80::429:fcff:fe86:31fa/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:125968 errors:0 dropped:0 overruns:0 frame:0
          TX packets:101137 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:13508626 (13.5 MB)  TX bytes:17795310 (17.7 MB)
```

or use the following command :

`curl ifconfig.co`

Results:
```console
# curl ifconfig.co
158.175.102.12
```

> This address should be same as the one used to ssh your VM.

Then, edit the hosts file with this command:

`nano /etc/hosts`

Results:
```console
127.0.0.1	localhost
127.0.1.1	myhostname.workshop	myhostname
...
```

> Note : you can see some IP v6 definitions - don't touch them.
> Don't touch the hostnames

Change the line containing **127.0.1.1** with your ipaddress :

**123.456.789.123**   myhostname.workshop	    myhostname     

> Save the file (ctrl O, enter, ctrl X )

Your /etc/hosts file should look like that after the change (please don't touch the hostnames):

```console
# more /etc/hosts
127.0.0.1	        localhost
123.456.789.123 	niceicp15.sl.workshop	niceicp15
...
```

Use the following 2 commands to **update** the system with some complementary packages:

```console
apt update
apt-get install apt-transport-https ca-certificates curl software-properties-common python-minimal jq
```

Check carefully the output to see if you have errors and especially if you cannot connect.

Increase your virtual memory maximum by typing the command:

`sysctl -w vm.max_map_count=262144`

![apt install](./images/Update2.png)


### Task 2 : Add Docker’s official GPG key

Use the following command:

```console
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
```

### Task 3: Add a repo to get the Docker

Use the following 2 commands:

```console
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

`apt-get update`

### Task 4: Get Docker

You can list all versions available in the added repo:

`apt-cache madison docker-ce`

Use the following command to get Docker installed (Docker version 17.12.1 is the maximum version supported by IBM Cloud Private 2.1.0.3):

`apt-get install docker-ce=17.12.1~ce-0~ubuntu`

![apt install](./images/InstallDocker.png)

Check that Docker is running (client and server):

`docker version`

![Check Docker is running](./images/CheckDocker.png)

> Docker client is mainly the CLI. The server is composed of different programs and among then, there is the **containerd**, the engine managing the isolation/running of containers.

You can look around at the docker CLI (and go thru some of  the sub-commands) by just typing:

`docker`

`docker ps`

`docker images`

Another lab will go in detail to explain how Docker is managing the images and running them.

### Task 5: Download IBM Cloud Private

We are going to use Docker to download the ICP-ce (community edition) package from the **dockerHub** web site:

`docker pull ibmcom/icp-inception:2.1.0.3`

IBM has packaged all the main components necessary for the installation in one package (note that you can change the version to a more recent one if needed). The pull sub-command is going to download the image on the local file system (the image will be run to install ICP).

![Download ICP-ce from Docker HUB](./images/DownloadICP.png)

Create a directory and move to that directory:

`mkdir /opt/icp`

`cd /opt/icp `  

Copy the ICP package into that directory:

```console
docker run -e LICENSE=accept -v "$(pwd)":/data ibmcom/icp-inception:2.1.0.3 cp -r cluster /data
```

> Note:  this docker command is executing the linux copy (cp) command from the volume (-v parameter). This will create a cluster directory in /opt/icp with all necessary files.


### Task 6: SSH Keys setup

We are going to generate new ssh keys in the /opt/icp/cluster directory:

`ssh-keygen -b 4096 -f ~/.ssh/id_rsa -N "" `

`cat ~/.ssh/id_rsa.pub | sudo tee -a ~/.ssh/authorized_keys `

`systemctl restart sshd `

`cp ~/.ssh/id_rsa ./cluster/ssh_key  `

> Finally a new key should now exist in the /opt/icp/cluster:

`ls /opt/icp/cluster`

![Generate new keys](./images/keys.png)

### Task 7: Customize ICP

Add the IP address of each node in the cluster to the **/opt/icp/cluster/hosts file** (in our case, we use the same ip address for each component).

> Use the unique **ipaddress** that you have already used.

`nano /opt/icp/cluster/hosts`

![ip configuration of ICP](./images/config-icp.png)

Save the file (ctrl O, enter, ctrl X ).

To understand the installation setup, you can look at the /opt/icp/cluster/config.yaml where you will find all the parameters.

`more /opt/icp/cluster/config.yaml`

No need to change for default installation.

> *hosts* and *config.yaml* are the 2 most important files to parameter the ICP installation. In a "standard configuration" with multiple worker nodes, you should specify a list of IP addresses under the worker section. At the time of this version, only ip addresses are supported in the *hosts* file.

**You are now ready to install IBM Cloud Private**.


### Task 7: Install ICP

Finally install ICP by using those commands:

`cd /opt/icp/cluster`

```console
docker run -e LICENSE=accept --net=host -t -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.3 install
```

Installation should last around 20 minutes. Check messages.
In case of error (red messages), make the appropriate change in the configuration files. Sometimes the installation can slowdown a little bit with **Retrying** messages  (during Cloudant or IAM startup for instance).

The end of the installation should look like this :

![End of the Installation](./images/EndInstall.png)

> Note that during a simple installation, there are **no failures and no errors**. If some errors occur and the install exits then first retry the install command and follow the instructions. If it doesn't work then "uninstall" ICP, correct the issue and restart the install process.

**In case of Uninstall**
```console
docker run -e LICENSE=accept --net=host -t -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.3 uninstall
```


**BEFORE GOING TO THE NEXT STEP, WAIT 5 MINUTES** so that everything can start gracefully.


Use the green link at the end of the installation script to get access to the console (admin/admin) in a browser:

`https://ipaddress:8443`  

![Loginicp](./images/loginicp.png)

You should receive the **Welcome Page**:

![Welcome to ICP](./images/Welcome.png)

Click on the **Catalog** menu (top right) to look at the list of applications already installed:

![Menu](./images/Hamburger.png)

The **Catalog** shows Charts that you can visit (it could take au few seconds to refresh the first time)

You can look at the (helm) catalog and visit some entries (but don't create any application at the moment).

### Task 8: Install and configure CLIs

At this point, you will need to install the **Kubernetes CLI** (command kubectl).  
For that purpose, open a ssh terminal with the Ubuntu VM in root mode and type the following command:

```console
docker run -e LICENSE=accept --net=host -v /usr/local/bin:/data ibmcom/icp-inception:2.1.0.3 cp /usr/local/bin/kubectl /data
```

This docker command will copy the kubectl program to the /usr/local/bin.

Now, you need to setup the endpoint to tell the kubectl command where is the ICP Cluster and what are the correct certificates.
To do so, go to the ICP console and select your profile on the right hand:

![Configure Client](./images/Client.png)

Click on Configure client:

![Configure Client](./images/ClientSetup.png)

These 5 lines contain a token that change every 12 hours. So then, you generally have to use these 5 commands to get connected.

> We are **not** going to use this method to connect to the cluster

We are going to create a script to help us to connect to our cluster.
Go back to the ssh or putty terminal :

`cd`

`nano connect2icp.sh`

Copy the following code (inspired from the 5 lines) :

```console
CLUSTERNAME=mycluster
ACCESS_IP=`curl ifconfig.co`
USERNAME=admin
PASSWD=admin

token=$(curl -s -k -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" -d "grant_type=password&username=$USERNAME&password=$PASSWD&scope=openid" https://$ACCESS_IP:8443/idprovider/v1/auth/identitytoken --insecure | jq .id_token | awk  -F '"' '{print $2}')

kubectl config set-cluster $CLUSTERNAME.icp --server=https://$ACCESS_IP:8001 --insecure-skip-tls-verify=true
kubectl config set-context $CLUSTERNAME.icp-context --cluster=$CLUSTERNAME.icp
kubectl config set-credentials admin --token=$token
kubectl config set-context $CLUSTERNAME.icp-context --user=admin --namespace=default
kubectl config use-context $CLUSTERNAME.icp-context
```

Save the file (ctrl O, enter, ctrl X ) and make this file executable :

`chmod +x connect2icp.sh`

Then execute that shell program :

`~/connect2icp.sh`

Results :

```console
# ~/connect2icp.sh
Cluster "cluster.local" set.
Context "cluster.local-context" modified.
User "admin" set.
Context "cluster.local-context" modified.
Switched to context "cluster.local-context".
```

As a result, you will see that you are now **connected** to the cluster (with only one node):

`kubectl version --short`

Results :

```console
# kubectl version --short
Client Version: v1.10.0
Server Version: v1.10.0+icp
```

Try this command to show all the worker nodes :

`kubectl get nodes`

Results :

```console
# kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
159.8.136.130   Ready     <none>    1h        v1.10.0+icp
```

> After a long period of inactivity, if you see some connection error when typing a kubectl command then re-execute the `~/connect2icp.sh` command.

To get help from the kubectl, just type this command:

`kubectl`

> **Optional** : it could be interesting to define an alias name for docker and kubectl commands.

To do so, from the ssh terminal, type the following commands:

`cd`

`nano .bashrc`

At the end of the file, add the 3 lines:

```console
alias k='kubectl'
alias d='docker'
alias h='helm'
```

Save the file (ctrl O, enter, ctrl X ). Source your script with the following command:

`source .bashrc`

Now test your new commands:

![Alias](./images/testK.png)

Finally, you can also install the **ic** command (former bx command) :

`curl -fsSL https://clis.ng.bluemix.net/install/linux | sh`

![bx CLI](./images/bxCLI.png)

the **ibmcloud** command is a generic command for all the IBM Cloud (private and public). The former name bx comes from Bluemix.

The ibmcloud command is using **plugins** to add some features. Now, download the ibmcloud pr **plugin** to get the ICP specific commands:

```console
wget https://ipaddress:8443/api/cli/icp-linux-amd64 --no-check-certificate
```

![Download the ICP plugin](./images/plugin.png)

Finally add this plugin to ibmcloud :

`ibmcloud plugin install icp-linux-amd64`

`ibmcloud plugin show icp`

![ICP Command plugin help](./images/pluginhelp.png)

Before using the **ibmcloud pr**, you must login to the master:

`ibmcloud pr login -a https://mycluster.icp:8443 --skip-ssl-validation`

> For the login : admin/admin

```console
# ibmcloud pr login -a https://mycluster.icp:8443 --skip-ssl-validation
API endpoint: https://mycluster.icp:8443

Username> admin

Password>
Authenticating...
OK

Select an account:
1. mycluster Account (id-mycluster-account)
Enter a number> 1
Targeted account mycluster Account (id-mycluster-account)

Configuring helm and kubectl...
Configuring kubectl: /root/.bluemix/plugins/icp/clusters/mycluster/kube-config
Property "clusters.mycluster" unset.
Property "users.mycluster-user" unset.
Property "contexts.mycluster-context" unset.
Cluster "mycluster" set.
User "mycluster-user" set.
Context "mycluster-context" created.
Switched to context "mycluster-context".

Cluster mycluster configured successfully.

Configuring helm: /root/.helm
Helm configured successfully

```

With that **ibmcloud pr** CLI, you can manage the infrastructure part of the cluster like adding new worker nodes (machine-type-add, worker-add) and so on.

Then you can type some commands concerning your cluster:

`ibmcloud pr masters mycluster`

Results
```console
# ibmcloud pr masters mycluster
ID                      Private IP        Machine Type   State   
mycluster-00000000-m1   159.122.190.251   -              deployed   
```

Among all sub-commands in 'ibmcloud pr', there are some commands to manage the infrastructure components like :
- cluster
- workers (adding, removing ...)
- register (docker image management )
- helm repositories


Finally also add the **ibmcloud dev** plugin to successfully use the **Microservice Builder**:

`ibmcloud plugin install dev -r Bluemix`

![Adding the bx dev plugin](./images/bxdev.png)

### Task 9: Adding persistent storage to Kubernetes

Go to the ICP management console and click on the menu (top left hand side).
Select **Dashboard** in the menu.


![Dashboard](./images/pv-dashboard.png)

Normally after the installation, all internal storage is used by ICP (so that's why we see 0 % available in the storage metric).
We are now going to add some persistent storage.
In the terminal, use the following commands:

`cd /tmp`

`mkdir data01`

Open a file with that command:

`nano pv-rwo.yaml`

Copy and paste the following text in the file (this is a kubernetes document):

```console
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv-once-test1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data01
  persistentVolumeReclaimPolicy: Recycle
```
Save the file (ctrl O, enter, ctrl X ).

With that document, we are telling Kubernetes to create a persistent storage of 1GB with read/write once mode in the host path /tmp/data01.

To create this persistent volume, just type the following command:

`kubectl create  -f ./pv-rwo.yaml`

![Persistent Storage Creation](./images/pv-create1.png)

Now create a 2GB persistent storage volume with read/write/many mode:

`nano pv-rwm.yaml`

Copy and paste the following text in the file (this is a kubernetes document):

```console
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv-many-test1
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 2Gi
  hostPath:
    path: /tmp/data01
  persistentVolumeReclaimPolicy: Recycle
```
Save the file (ctrl O, enter, ctrl X ).
And then use the following command:

`kubectl create  -f ./pv-rwm.yaml`

Once created these 2 volumes (hostpath) should be listed with the following command:

`kubectl get pv`

![Persistent Volume](./images/pv-volumes.png)

> Note: the 2 volumes are shown as available storage.

Now go back to the Dashboard to see the change:

![Shared Storage](./images/pv-dashboard2.png)



### Task 10: End of installation

At this point, you can now go thru some other labs to implement applications using containers and Kubernetes solutions.


# 5. Second Option: Using Vagrant and VirtualBox

This option is an alternative to the previous (simple) detailed installation. We are not covering this installation for the moment but you can try it on your own laptop.
This ICP installation uses VirtualBox (that you need to install as a prerequisites).

The link below explain the steps:

https://github.com/IBM/deploy-ibm-cloud-private/blob/master/docs/deploy-vagrant.md

Note: this installation is using ICP version 2.1.0.1 instead of the latest one. You should change the version to **2.1.0.3**in the configuration file. Also don't stop the VM and use the vagrant suspend command.


# 6. Conclusion

###  Results
<span style="background-color:yellow;">Successful exercise ! </span>
You finally went thru the following features :
- [x] You setup a VM using Ubuntu version 16.04
- [x] You checked all the prerequisites before the ICP installation
- [x] You installed Docker
- [x] You installed ICP community edition (version 2.1.0.3) on one host
- [x] You connected to the ICP console
- [x] You setup the kubectl command line
- [x] You setup some persistent storage
- [x] You installed a functional Kubernetes Cluster on a single Node for testing and learning

---
# End of Lab
---

#  appendix A : Preparing your VM (VMWare) with Ubuntu

###  Task A1 - Download Ubuntu

Download the Ubuntu server **version 16.04.04** (ubuntu-16.04.4-server-amd64.iso) on your laptop.

###  Task A2 - Create a VM

Open VMWare Workstation or VMWare Fusion depending if you are running Windows or MacOS. Create a **new custom VM** with the following **customized** settings:
- Ubuntu 64 bits
- 8 cores
- 10 GB of RAM
- Change to 80 GB of storage in the hard drive
- NAT (or Share with my Mac) for the Network
- Specify the downloaded file in the DVD (ubuntu-16.04.4-server-amd64.iso) and check "connect the DVD Drive"
- Save the settings and start the VM

###  Task A3 - Set the language

Select English as the language by default

![Select English](./images/Select English.png)


###  Task A4 - Set the keyboard

Select your keyboard by **typing F3** and then choose a language for your Keyboard

![Select Keyboard](./images/Keyboard.png)

###  Task A5 - Install Ubuntu Server

Press enter to install Ubuntu server

![Select Install](./images/Install.png)

Press enter to use English as the language by default

![Select Install](./images/Language.png)

###  Task A6 - Select Location

Select your location (Other>Europe>France for example)

![Select Location](./images/Location.png)

###  Task A7 - Select locales

Select your locales (Other>Europe>France for example)

![Select Locales](./images/Locales.png)

###  Task A8 - Choose a hostname

Change the hostname of leave the default (ubuntu)

![Change the hostname](./images/Hostname.png)

###  Task A9 - Choose a username

Choose a user name (phil in my example)

![Change the user name](./images/user.png)

###  Task A10 - Type a password

Choose a user name (phil in my example) and choose a password (password) and do not encrypt the home directory

![Change the user name](./images/user.png)

![Change the password](./images/password.png)

###  Task A12 - Partition your disk

Choose "use entire disk" and accept the proposed partitioning

![Change to use entire disk](./images/Storage.png)

![Partition Disks](./images/Partition Disks.png)

###  Task A13 - No Proxy

Leave the HTTP proxy line empty
(unless you're using a proxy server to connect to the Internet)

![Leave proxy to empty line](./images/proxy.png)

###  Task A14 - Launch the installation

I like to update my servers automatically. Therefore, I select Install Security
Updates automatically

![select automatic security patches](./images/security.png)

###  Task A15 - Add some packages

Select the following items : **OpenSSH server**, **Standard System
Utilities**, **manual package selection** and **Virtual Host** so that I can immediately connect to the system with a SSH client such as PuTTY after the installation has finished

![select packages](./images/packages.png)

###  Task A16 - GRUB

Accept to install GRUB and this will end the installation process with a **reboot** of the VM :

![accept Grub](./images/Grub.png)

###  Task A17 - Login for the first time

First login with  your credentials (that you defined during the installation) **from the VMware console**

![First Login](./images/First Login.png)

###  Task A18 - Get the IP address

Get the IP address of this virtual machine by typing:

`ifconfig`

In that case look at the Ethernet interface and get the inet addr : 192.168.225.132 (for example). Collect also the Bcast and Mask.

![Get the ip](./images/ip.png)

###  Task A19 - SSH to the VM

Connect by using ssh (the user name and the ip address):

``` ssh phil@192.168.225.132 ```

Enter yes and then your password

![First Login](./images/ssh.png)

Set a password to root by using the following command:

`sudo -i`

`passwd`

![set a password to root](./images/root.png)

###  Task A20  - Update the OS

Update the Ubuntu VM. Type the following command (be sure to be root):

`apt update`

![update the image](./images/Update.png)

###  Task A21 - Define the network interface

Update the network interface:

`nano /etc/network/interfaces`

Modify the primary interface (in this example ens33) by adding some static definitions :

> auto ens33
>
> iface ens33 inet static
>
> address 192.168.225.7
>
> netmask 255.255.255.0
>
> network 192.168.225.0
>
> broadcast 192.168.225.255
>
> gateway 192.168.225.2
>
> dns-nameservers 192.168.225.2 8.8.8.8



Be careful:  the interface name, the IP address, the gateway address and netmask could be different (refer to the ifconfig command to get the right names and IPs)

![update the interfaces](./images/interfaces.png)

Save the file (ctrl O, enter, ctrl X )

Also edit the ssh config to authorize the root login:

`nano /etc/ssh/sshd_config`

![authorize root login in ssh](./images/rootlogin.png)

Save the file (ctrl O, enter, ctrl X )

**REBOOT YOUR VM**:

`shutdown -r 0`

After reboot, reconnect (ssh) to your VM with root credentials

###  Task A22 - You are now ready to start the ICP installation

> Go to section 4.


#  appendix B : Changing ICP admin password

This tutorial describes how to change the admin password.

> ATTENTION : This procedure could be dangerous - Knowing **vi** is a prerequisite.

###  Task B1 - Login to your ICP cluster using ssh

`ssh root@ipaddress`

###  Task B2 - Generate your new ICP password in base64

`echo -n "MyNewPassword"| base64`

Results :
 ```console
 # echo -n "MyNewPassword"| base64
TXlOZXdQYXNzd29yZA==
 ```
> Attention choose a very specific password to your ICP (containing capital letters and numbers and special characters ...)
> **Take a note of the  encrypted password**

###  Task B3 - Edit ICP secrets

`kubectl -n kube-system edit secrets platform-auth-idp-credentials`

Results :
```console
# kubectl -n kube-system edit secrets platform-auth-idp-credentials
error: You must be logged in to the server (Unauthorized)
```

> If you see that message then use the ./connect2ICP.sh to reconnect to the cluster.

`kubectl -n kube-system edit secrets platform-auth-idp-credentials`

Results :
![vi secrets](./images/visecrets.png)

This command opens up the **vi** text editor on the secrets file.
Locate the **admin-password** and change the existing encrypted password with the one that you generated.
Don't change anything else in the file.
Save your work : **escape  :wq!**

###  Task B4 - Delete the auth pods

`kubectl -n kube-system delete pods -l k8s-app=auth-idp`


Results :

```console
# kubectl -n kube-system delete pods -l k8s-app=auth-idp
pod "auth-idp-sv8gv" deleted
```

###  Task B5 - Test the new password

Wait until the auth pods have been restarted

`kubectl -n kube-system get pods -l k8s-app=auth-idp`

Results :

```console
# kubectl -n kube-system get pods -l k8s-app=auth-idp
NAME             READY     STATUS    RESTARTS   AGE
auth-idp-swgrc   3/3       Running   0          46s
```
Then test the new password :

**First test : go to the ICP console http://ipaddress:8443 and log in with your new password. If it works, modify also connect2icp.sh script**

`cd`

`nano ./connect2icp.sh`

Change the 4th line PASSWD with your new password.


![new password](./images/newpassword.png)



Save the file (Ctrl+o, Enter, Crtl+x)

Execute the command :

`./connect2icp.sh`

Results:
```console
# ./connect2icp.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    16  100    16    0     0     69      0 --:--:-- --:--:-- --:--:--    69
Cluster "mycluster.icp" set.
Context "mycluster.icp-context" modified.
User "admin" set.
Context "mycluster.icp-context" modified.
Switched to context "mycluster.icp-context".
```




---
# End of Appendix
---

<div style="background-color:black;color:white; vertical-align: middle; text-align:center;font-size:250%; padding:10px; margin-top:100px"><b>
 Installing IBM Cloud Private
 </b></a></div>
