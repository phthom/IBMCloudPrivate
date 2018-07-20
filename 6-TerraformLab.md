


<div style="background-color:black;color:white; vertical-align: middle; text-align:center;font-size:250%; padding:10px; margin-top:100px"><b>
IBM Cloud Private - Terraform Lab
 </b></a></div>


---
# Terraform Lab
---


![](./images/Terraform-logo.png)


Terraform is a tool for building, changing, and versioning **infrastructure** safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

Configuration files describe to Terraform the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.

The infrastructure Terraform can manage includes low-level components such as compute instances, storage, and networking, as well as high-level components such as DNS entries, SaaS features, etc.

**In this tutorial, you will use Terraform to create a new VM in the _IBM Cloud_ Infrastructure so that you can automatically install IBM Cloud Private in a  standalone cluster.** 

> **IMPORTANT PREREQ** : you should have an IBM Cloud Infrastructure (SoftLayer) account to execute this lab.


## Table of Contents

---
- [Task 1: Getting IBM Cloud Infra information](#task-1--getting-ibm-cloud-infra-information)
- [Task 2: Installing Terraform](#task-2--installing-terraform)
- [Task 3: Installing the plugin and CLI](#task-3--installing-the-plugin-and-cli)
- [Task 4: SSH Keys](#task-4--ssh-keys)
- [Task 5: Summary](#task-5--summary)
- [Task 6: Define your main.tf and variables.tf](#task-6--define-your-maintf-and-variablestf)
- [Task 7: Terraform Plan](#task-7--terraform-plan)
- [Task 7: Terraform Apply](#task-7--terraform-apply)
- [Task 8: Using script to configure the VM](#task-8--using-script-to-configure-the-vm)
- [Congratulations](#congratulations)
---
 
 
# Task 1: Getting IBM Cloud Infra information


We need to collect your IBM Cloud Infrastructure (SoftLayer) Username and API Key.

Log in to the **SoftLayer Control page** https://control.softlayer.com/

Look at the top right and note which account you're logged in as. You may have more than one account. 
Click on your name to edit your profile :

Scroll down to the bottom and look for the section API Access Information
You will see an "API Username" and "Authentication key" section. The "API Username" will be something like 9999999_shortname@us.ibm.com or a name like IBM99999

For example

---
```console
API Access Information
API Username
IBM709789

Authentication Key
ea0b807aa8c96273aa1e25f0cdffe1bd69f37729b849408a9981c9decf3790ed
```
---

Take a note on both API Username and Authentication Key.


# Task 2: Installing Terraform

Install Terraform CLI from this page :

https://www.terraform.io/downloads.html

# Task 3: Installing the plugin and SoftLayer CLI

Install the Terraform provider for SoftLayer :

https://github.com/softlayer/terraform-provider-softlayer/releases

Install the SoftLayer CLI :

http://softlayer-api-python-client.readthedocs.io/en/latest/install/

Configure the CLI with the SoftLayer Username and API Key :

`slcli setup`

Results :
```console
# slcli setup
Username [IBM709890]: 
API Key or Password [c96ea0f3c9decf370cdffe1bd69f90b807aa8273aa1e27729b849408a99815ed]: 
Endpoint (public|private|custom) [public]: 
Timeout [0]: 
:..............:..................................................................:
:         name : value                                                            :
:..............:..................................................................:
:     Username : IBM709990                                                        :
:      API Key : c9669f3c9deea0b807aa8273aa1408a99815f0ce27729b849dffe1bdcf3790ed :
: Endpoint URL : https://api.softlayer.com/xmlrpc/v3.1/                           :
:      Timeout : not set                                                          :
:..............:..................................................................:
Are you sure you want to write settings to "/Users/phil/.softlayer"? [Y/n]: yes
Configuration Updated Successfully
```

# Task 4: SSH Keys

Now you will create an ssh key pair and register it with Softlayer.
The name you give the key in Softlayer has to be unique. This can be tricky if you are using a shared account. You can't just call it ssh-key.
For now, use your name and some numbers  to make the key ID unique. For the following steps, replace 'shortname_ssh_key' with your own actual key ID.

> On windows, you may need to install OpenSSH separately (check `ssh` command in the CLI ).

On the command line, enter the following commands :
```console
cd
mkdir terra
cd terra
ssh-keygen -f shortname_ssh_key -P ""
```

You will end up with two files in the current directory, shortname_ssh_key (the private key) and shortname_ssh_key.pub (the public key)
Next, you'll register the public key with Softlayer by executing :

`slcli sshkey add -f shortname_ssh_key.pub shortname_ssh_key`

Then list all your ssh keys with the following command :

`slcli sshkey list`

Results :

```console
:.........:....................:.................................................:.......:
:    id   :       label        :                   fingerprint                   : notes :
:.........:....................:.................................................:.......:
:  920557 :  Temp Public Key   : ba:18:fc:54:82:ee:f3:47:a2:be:91:24:72:fa:d4:b4 :   -   :
: 1146085 :  Temp Public Key   : 37:cd:69:ef:08:7c:42:36:92:31:a3:33:e9:38:45:eb :   -   :
:  842075 :  Temp Public Key   : 1a:83:27:96:d7:ab:8f:bf:af:21:eb:b7:28:cd:87:71 :   -   :
:  842077 :   CAM Public Key   : 94:4d:f0:8b:89:72:61:7f:a9:9a:87:b4:7a:a5:6c:b7 :   -   :
:  920445 :   CAM Public Key   : b3:c4:47:8c:cb:dc:50:dc:27:11:d9:b8:15:15:57:55 :   -   :
:  921603 :  Temp Public Key   : f1:63:cd:ff:a9:22:d1:b5:8b:61:c8:c4:41:0a:75:a4 :   -   :
: 1150051 :  Temp Public Key   : 02:b2:66:e4:52:e4:ef:ce:ab:1c:49:08:f5:ac:a9:35 :   -   :
: 1145559 : Orpheus Public Key : f6:3b:9a:7c:02:6f:a6:31:f0:1b:78:3a:c0:a5:59:ec :   -   :
: 1145561 :  Temp Public Key   : 06:ea:10:41:1d:53:c1:42:22:32:fd:4e:6d:41:ab:f9 :   -   :
: 1147633 : shortname_ssh_key  : 85:8a:95:9d:ce:61:0d:8d:c5:01:cc:34:63:7d:76:66 :   -   :
: 1150085 :   CAM Public Key   : 35:c6:3e:20:cc:ad:4c:7e:59:05:d7:ba:15:53:2f:cb :   -   :
: 1147811 :     SL017487K      : 1a:cc:a3:14:2a:54:cd:2b:39:2f:de:6b:9a:c7:97:df :   -   :
:.........:....................:.................................................:.......:
```

  Locate your **shortname_ssh_key** and take a note of the sshkey ID (**1147633**).


# Task 5: Summary

Now you have collected the following information :

- SoftLayer (IBM Cloud Infra) **Username** 
- Authentication **APIKey**
- **SSHKEYID** that will be used for creating VMs.


# Task 6: Define your main.tf and variables.tf

If you are not in the terra directory, then move to it :

`cd terra`

Create a new file called **main.tf**
The main.tf file is a template that will contain all the VM definitions with variables.
Here is an example (just change the **Username** and **APIKey** with you own values) :
 
```console
provider "softlayer" {
  username = "Phil"
  api_key  = "hhohhiohiohT7876767868Ykbkbkjkjjkbbk"
}

resource "softlayer_virtual_guest" "icpw" {
      count       = "${var.VM["nodes"]}"

      datacenter  = "${var.datacenter}"
      domain      = "${var.domain}"
      hostname    = "${format("${lower(var.instance_name)}%01d", count.index + 1) }"
      os_reference_code = "UBUNTU_16_64"
      cores       = "${var.VM["cpu_cores"]}"
      memory      = "${var.VM["memory"]}"
      disks       = ["${var.VM["disk_size"]}"]
      local_disk  = "${var.VM["local_disk"]}"
      network_speed         = "${var.VM["network_speed"]}"
      hourly_billing        = "${var.VM["hourly_billing"]}"
      private_network_only  = "${var.VM["private_network_only"]}"
      ssh_key_ids = ["${var.VM["ssh_key_id"]}"]
  }
```


Then create anther file called **variables.tf**. 
In that file (see the example below). Change the datacenter default (**lon02**) with the one  of your choice. Specify a new **hostname** (or instance name).Finally, specify the number of VMs that you want and all the parameters (cpu_size, memory, cores ...) including the last one (**SSHKEYID**). Don't forget that **nodes** represents the number of VMs that you want to order to IBM.

```console
##### Common VM specifications ######
variable "datacenter" { default = "lon02" }         # ATTENTION : datacenter
variable "domain" { default = "ibm.ws" }

##### Start Name
variable "instance_name" { default = "myhostname" }     # ATTENTION : hostname

##### VSI Instance details ######
variable "VM" {
  type = "map"
  default = {
    nodes                 = "1"                     # ATTENTION : number of VMs
    cpu_cores             = "8"
    disk_size             = "100"
    local_disk            = false
    memory                = "16384"
    network_speed         = "1000"
    private_network_only  = false
    hourly_billing        = true
    ssh_key_id            = "1147633"
  }
}

```



# Task 7: Terraform Plan

This step is about checking that everything is ok for this first request :

In the directory where you defined the 2 previous files, type :

`terraform plan`

Results :
```console
# terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + softlayer_virtual_guest.icpw
      id:                       <computed>
      cores:                    "8"
      datacenter:               "lon02"
      disks.#:                  "1"
      disks.0:                  "100"
      domain:                   "ibm.ws"
      hostname:                 "myhostname"
      hourly_billing:           "true"
      ip_address_id:            <computed>
      ip_address_id_private:    <computed>
      ipv4_address:             <computed>
      ipv4_address_private:     <computed>
      ipv6_address:             <computed>
      ipv6_address_id:          <computed>
      ipv6_enabled:             "false"
      local_disk:               "false"
      memory:                   "16384"
      network_speed:            "1000"
      os_reference_code:        "UBUNTU_16_64"
      private_network_only:     "false"
      private_subnet:           <computed>
      private_vlan_id:          <computed>
      public_ipv6_subnet:       <computed>
      public_subnet:            <computed>
      public_vlan_id:           <computed>
      secondary_ip_addresses.#: <computed>
      ssh_key_ids.#:            "1"
      ssh_key_ids.0:            "1147633"
      wait_time_minutes:        "90"


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.

```

If there is no error then go to the next step.



# Task 7: Terraform Apply

This step will execute the order in SoftLayer.

`terraform apply`

After **5 minutes**, your VM will be ready.

To show the VMs, type :

`slcli vs list`

Results :

```console
slcli vs list
:..........:..............................:.................:................:............:........:
:    id    :           hostname           :    primary_ip   :   backend_ip   : datacenter : action :
:..........:..............................:.................:................:............:........:
: 38556487 :            GeWeb1            :   159.8.253.23  :  10.137.7.229  :   ams03    :   -    :
: 52693275 :          dzcomposer          :  169.51.44.146  : 10.175.71.100  :   ams03    :   -    :
: 24843259 :       ehn-vsp-rhel6-1        :        -        : 10.134.181.163 :   fra02    :   -    :
: 25035053 :       ehn-vsp-u16lts-2       :        -        : 10.134.181.157 :   fra02    :   -    :
: 25049253 :       ehn-vsp-u16lts-3       :        -        : 10.134.181.181 :   fra02    :   -    :
: 23077293 :       ehn-vsp-w12r2-1        :        -        : 10.134.181.155 :   fra02    :   -    :
: 24565387 :            frank1            :        -        : 10.134.181.154 :   fra02    :   -    :
: 24610715 :            myhostname        :  169.51.44.146  : 10.134.181.144 :   fra02    :   -    :

```

From this list, locate your VM and display the root password :

` slcli vs detail myhostname --passwords`

Results :

```console
slcli vs detail nicetest1 --passwords
:....................:......................................:
:               name : value                                :
:....................:......................................:
:                 id : 56755031                             :
:               guid : bd2d39fd-05f7-4a6e-a1c4-0e7867655175 :
:           hostname : myhostname                           :
:             domain : ibm.ws                               :
:               fqdn : myhostname.ibm.ws                    :
:             status : Active                               :
:              state : Running                              :
: active_transaction : -                                    :
:         datacenter : mil02                                :
:                 os : Ubuntu                               :
:         os_version : 16.04-64 Minimal for VSI             :
:              cores : 8                                    :
:             memory : 16G                                  :
:          public_ip : 169.51.44.146                        :
:         private_ip : 10.124.112.232                       :
:       private_only : False                                :
:        private_cpu : False                                :
:            created : 2018-06-27T08:50:29+01:00            :
:           modified : 2018-06-27T08:56:29+01:00            :
:              owner : IBM708531                            :
:              vlans : :.........:........:.........:       :
:                    : :   type  : number :    id   :       :
:                    : :.........:........:.........:       :
:                    : :  PUBLIC :  901   : 2356505 :       :
:                    : : PRIVATE :  1166  : 1657599 :       :
:                    : :.........:........:.........:       :
:              users : :..........:..........:..........:   :
:                    : : software : username : password :   :
:                    : :..........:..........:..........:   :
:                    : :  Ubuntu  :   root   : Vhiu572j :   :
:                    : :..........:..........:..........:   :
:               tags : -                                    :
:....................:......................................:
```
Then you can connect to the VM either using the root/password or the sshkey (or use putty to get access on Windows)

`ssh root@ipaddress`

> Note : to delete a VM, use the following command : `slcli vs cancel vm-id` where the vm-id is the number found on the list of VMs.

# Task 8: Using scripts to configure the VM

The VM could be configured and customized after its creation. 
A script can be copied and launched when Terraform finished launching the VM. 
In our case, we want to **install IBM Cloud Private** in the VM. 

Go to the terra directory we created before.
Create a new file called **icpinstall.sh**

```console
#!/bin/bash
#
# THIS SCRIPT ONLY WORKS FOR LINUX
# either run as root or as a sub-account with sudo configured for NOPASSWORD prompt
# usage: ./icpinstall.sh
# If you see the following error Error retrieving SSH key: SOAP-ENV:Client: Bad Request (HTTP 200)
# comment out the line beginning with endpoint_url from ~/.softlayer.

echo "***"
echo "*** START"
echo "***"


MASTERIP=`curl ifconfig.co`
echo "***"
echo "*** $MASTERIP"
echo "***"

HOSTNAME=`hostname`
echo "***"
echo "*** $HOSTNAME"
echo "***"

# Update system
echo "*** Upadate System"
apt-get -q update

# Update /etc/hosts
echo "*** Update /etc/hosts"

echo "127.0.0.1 localhost" > /etc/hosts
echo "$MASTERIP $HOSTNAME.ibm.ws $HOSTNAME" >> /etc/hosts

# Add some modules
echo "*** Add some modules"
apt-get -y install apt-transport-https ca-certificates curl software-properties-common python-minimal jq
sysctl -w vm.max_map_count=262144
sed -i '/ swap / s/^/#/' /etc/fstab
echo "vm.max_map_count=262144" >> /etc/sysctl.conf

# Docker GPG key
echo "*** Docker GPG key"
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

# Repo for docker
echo "*** Repo for docker"
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get -q update

# Get Docker
echo "*** Get Docker"
apt-get -y install docker-ce=17.12.1~ce-0~ubuntu
docker version

# Get inception
echo "*** Get inception"
docker pull ibmcom/icp-inception:2.1.0.3

# Copy cluster
echo "*** Create Directory"
mkdir /opt/icp
cd /opt/icp

docker run -e LICENSE=accept -v "$(pwd)":/data ibmcom/icp-inception:2.1.0.3 cp -r cluster /data

# SSH aKeys
echo "*** Get SSH aKeys"
ssh-keygen -b 4096 -f ~/.ssh/id_rsa -N ""
cat ~/.ssh/id_rsa.pub | sudo tee -a ~/.ssh/authorized_keys
systemctl restart sshd
cp ~/.ssh/id_rsa ./cluster/ssh_key

# Customize hosts
echo "*** Customize hosts"
echo "[master]" > /opt/icp/cluster/hosts
echo "$MASTERIP" >> /opt/icp/cluster/hosts
echo " " >> /opt/icp/cluster/hosts
echo  "[worker]" >> /opt/icp/cluster/hosts
echo "$MASTERIP" >> /opt/icp/cluster/hosts
echo " " >> /opt/icp/cluster/hosts
echo "[proxy]" >> /opt/icp/cluster/hosts
echo "$MASTERIP" >> /opt/icp/cluster/hosts
echo " " >> /opt/icp/cluster/hosts
echo "[management]" >> /opt/icp/cluster/hosts
echo "$MASTERIP" >> /opt/icp/cluster/hosts
echo " " >> /opt/icp/cluster/hosts

# Install ICP
echo "*** Install ICP"
cd /opt/icp/cluster
docker run -e LICENSE=accept --net=host -t -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.3 install


echo "*** Install Kubectl"
docker run -e LICENSE=accept --net=host -v /usr/local/bin:/data ibmcom/icp-inception:2.1.0.3 cp /usr/local/bin/kubectl /data


echo "*** Connect to Cluster"

cd /root
./connect2icp.sh

echo "*** Install ic"
curl -fsSL https://clis.ng.bluemix.net/install/linux | sh
wget https://mycluster.icp:8443/api/cli/icp-linux-amd64 --no-check-certificate
ibmcloud plugin install icp-linux-amd64
ibmcloud plugin install dev -r Bluemix

echo "*** Install PV"
cd /tmp
mkdir data01

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv-once-test1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 30Gi
  hostPath:
    path: /tmp/data01
  persistentVolumeReclaimPolicy: Recycle
EOF

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv-many-test1
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 50Gi
  hostPath:
    path: /tmp/data01
  persistentVolumeReclaimPolicy: Recycle
EOF
echo "*** "
echo "*** End of Install"

```
Save this file.

We also need a connection script to get access to the cluster.
Go to the terra directory we created before.
Create a new file called **connect2icp.sh**

```console
#!/bin/bash
#
# THIS SCRIPT ONLY WORKS FOR LINUX
# either run as root or as a sub-account with sudo configured for NOPASSWORD prompt
# usage: ./connect2icp.sh
# Connect the user to the ICP Cluster 

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


Save this file.

Modify the **main.tf** file. We added 3 sections called **provisioners** : the first 2 ones copy the icpinstall.sh and connect2icp.sh files to the /root directory in the newly created VM and the third provisioner will execute the installation script.

```console
provider "softlayer" {
  username = "IBM708531"
  api_key  = "c96ea0b807aa8273aa1e27729b849408a99815f0cdffe1bd69f3c9decf3790ed"
}

resource "softlayer_virtual_guest" "icpw" {
      count       = "${var.VM["nodes"]}"

      datacenter  = "${var.datacenter}"
      domain      = "${var.domain}"
      hostname    = "${format("${lower(var.instance_name)}%01d", count.index + 1) }"
      os_reference_code = "UBUNTU_16_64"
      cores       = "${var.VM["cpu_cores"]}"
      memory      = "${var.VM["memory"]}"
      disks       = ["${var.VM["disk_size"]}"]
      local_disk  = "${var.VM["local_disk"]}"
      network_speed         = "${var.VM["network_speed"]}"
      hourly_billing        = "${var.VM["hourly_billing"]}"
      private_network_only  = "${var.VM["private_network_only"]}"
      ssh_key_ids = ["${var.VM["ssh_key_id"]}"]


provisioner "file" {
        source      = "./icpinstall.sh"
        destination = "/root/icpinstall.sh"

        connection {
            type     = "ssh"
            user     = "root"
            private_key = "${file("shortname_ssh_key")}"
          }
      }

provisioner "file" {
              source      = "./connect2icp.sh"
              destination = "/root/connect2icp.sh"

              connection {
                  type     = "ssh"
                  user     = "root"
                  private_key = "${file("shortname_ssh_key")}"
                }
            }

provisioner "remote-exec" {
        inline = [
          "chmod +x /root/icpinstall.sh",
          "chmod +x /root/connect2icp.sh",
          "/root/icpinstall.sh"
        ]
        connection {
            type     = "ssh"
            user     = "root"
            private_key = "${file("shortname_ssh_key")}"
          }
      }



  }

```
> Note : the 3 provisioners are  associated to the resource (i.e. the VM). Notice that you also have to replace the **shortname_ssh_key** with the name you choose.

To avoid a common error `Error retrieving SSH key: SOAP-ENV:Client: Bad Request (HTTP 200)`, then on your **laptop** : 

Go to your personal directory and visualize this file : 

`more ~/.softlayer`

```console 
# more .softlayer
[softlayer]
username = IBM975455
api_key = c96772f0cdffe1bd69f3c9d9b849408a99815ecf3ea0b807aa8273aa1e2790ed
timeout = 0
endpoint_url = https://api.softlayer.com/xmlrpc/v3.1/
```

Then comment out the endpoint_url (the last line) :

```console 
# more .softlayer
[softlayer]
username = IBM975455
api_key = c96772f0cdffe1bd69f3c9d9b849408a99815ecf3ea0b807aa8273aa1e2790ed
timeout = 0
# endpoint_url = https://api.softlayer.com/xmlrpc/v3.1/
```


Save the modifications and redo the `terraform plan` and `terraform apply`.
This will automatically create the VM and launch the IBM Cloud Private installation (you will see the same steps as usually).

Test the access to the IBM Cloud Private console :

https://ipaddress:8443

**Note** : to delete a VM, use the following command : 

`slcli vs cancel vm-id` 

where the vm-id is the number found on the list of VMs.

# Congratulations 

You have successfully used **Terraform**, the ultimate open source tool for using **Infrastructure as code** (IaC) to create a VM in IBM Cloud Infrastructure and you have been able to install automatically **IBM Cloud Private**.


----


<div style="background-color:black;color:white; vertical-align: middle; text-align:center;font-size:250%; padding:10px; margin-top:100px"><b>
IBM Cloud Private - Terraform Lab
 </b></a></div>
