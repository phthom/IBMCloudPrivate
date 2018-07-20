
---
# Getting access to a shell in a Container
---

### Prerequisites

You should be connected to you Kubernetes Cluster. 
>Note : this exercise is based on Kubernetes.io example resources. 

# Task 1: Create a Pod

In this exercise, you create a Pod that has one Container. The Container runs the nginx image. 

Create the Pod:

`kubectl create -f https://k8s.io/examples/application/shell-demo.yaml`

```
# kubectl create -f https://k8s.io/examples/application/shell-demo.yaml
pod "shell-demo" created
```

Verify that the Container is running:

`kubectl get pod shell-demo`

```
# kubectl get pod shell-demo
NAME         READY     STATUS    RESTARTS   AGE
shell-demo   1/1       Running   0          40s
```

Get a shell to the running Container:

`kubectl exec -it shell-demo -- /bin/bash`

```
# kubectl exec -it shell-demo -- /bin/bash
root@shell-demo:/# 
```

In your new shell (root@sell-demo), list the root directory:

`root@shell-demo:/# ls -l /` 

```
root@shell-demo:/# ls -l /
total 64
drwxr-xr-x   2 root root 4096 Jul 16 00:00 bin
drwxr-xr-x   2 root root 4096 Jun 26 12:03 boot
drwxr-xr-x   5 root root  360 Jul 20 08:47 dev
drwxr-xr-x   1 root root 4096 Jul 20 08:47 etc
drwxr-xr-x   2 root root 4096 Jun 26 12:03 home
drwxr-xr-x   1 root root 4096 Jul 16 00:00 lib
drwxr-xr-x   2 root root 4096 Jul 16 00:00 lib64
drwxr-xr-x   2 root root 4096 Jul 16 00:00 media
drwxr-xr-x   2 root root 4096 Jul 16 00:00 mnt
drwxr-xr-x   2 root root 4096 Jul 16 00:00 opt
dr-xr-xr-x 510 root root    0 Jul 20 08:47 proc
drwx------   2 root root 4096 Jul 16 00:00 root
drwxr-xr-x   1 root root 4096 Jul 20 08:47 run
drwxr-xr-x   2 root root 4096 Jul 16 00:00 sbin
drwxr-xr-x   2 root root 4096 Jul 16 00:00 srv
dr-xr-xr-x  13 root root    0 Jul  9 17:14 sys
drwxrwxrwt   1 root root 4096 Jul 17 04:21 tmp
drwxr-xr-x   1 root root 4096 Jul 16 00:00 usr
drwxr-xr-x   1 root root 4096 Jul 16 00:00 var
```

In your new shell, experiment with other commands. Here are some examples:

```
root@shell-demo:/# cat /proc/mounts
root@shell-demo:/# cat /proc/1/maps
root@shell-demo:/# apt-get update
root@shell-demo:/# apt-get install -y tcpdump
root@shell-demo:/# tcpdump
root@shell-demo:/# apt-get install -y lsof
root@shell-demo:/# lsof
root@shell-demo:/# apt-get install -y procps
root@shell-demo:/# ps aux
root@shell-demo:/# ps aux | grep nginx
```

# Task 2: Writing the root page for nginx
Look again at the configuration file for your Pod. The Pod has an emptyDir volume, and the Container mounts the volume at /usr/share/nginx/html.

In your shell, create an index.html file in the /usr/share/nginx/html directory:

`echo 'Hello shell demo' > /usr/share/nginx/html/index.html`

In your shell, add the curl command : 

```
apt-get update
apt-get install curl
````

Then send a GET request to the nginx server:

`curl localhost`

The output shows the text that you wrote to the index.html file:

```
Hello shell demo
````

When you are finished with your shell, enter exit.

`exit`

# Task 3: Running individual commands in a Container

In an ordinary command window, not in your shell, list the environment variables in the running Container:

`kubectl exec shell-demo env`

Experiment running other commands. Here are some examples:

```
kubectl exec shell-demo ps aux
kubectl exec shell-demo ls /
kubectl exec shell-demo cat /proc/1/mounts
```

> Note : Opening a shell when a Pod has more than one Container
If a Pod has more than one Container, use --container or -c to specify a Container in the kubectl exec command. For example, suppose you have a Pod named my-pod, and the Pod has two containers named main-app and helper-app. The following command would open a shell to the main-app Container.

`kubectl exec -it my-pod --container main-app -- /bin/bash`

## End of Lab

