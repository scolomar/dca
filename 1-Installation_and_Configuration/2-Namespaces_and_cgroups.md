## Chapter 1: Installation and Configuration

### Section 2: Namespaces and cgroups

Namespaces and cgroups constitute the core of the underlying technology behind containers.

Linux namespaces are a feature of the Linux Kernel that was initially released in 2002.
They are the base for containers.
In fact they actually constitute what we know as containers.

Namespaces are partitions of the resources of the Linux Kernel so that a set of resources such as for example the process identifier (PID) is assigned to a specific Linux namespace and is only accessible from inside that namespace.
In this case PID namespaces are nested meaning that from the original namespace you will be able to view the PID of any other process running in a nested namespace but you will not view the PID of any other process running outside your own namespace.
You have different types of namespaces:

* PID: For the process identifier (PID). 
This will isolate the PID of processes running in different namespaces. 
You will not have visibility of the PID of any process running outside your container.
* MNT: For the filesystem mount points.
This namespace will allow for isolation of the filesystem mounted inside a container from any other neighbouring container.
* NET: For the network stacks.
This namespace will provide visibility of the network stack for a specific container and totally isolate that network stack from any other container.
This will also allow for overlapping the range of IPs inside different containers.
The routing table will also be private to that container or network namespace.
Neutron uses this same technology to provide network isolation for the Openstack platform.
* IPC: For the Inter Process Communications.
Processes running inside a container can this way safely use the shared memory resources of the host machine and those resources will be private to the container or IPC namespace.
* UTS: For the Unix Time-Sharing namespaces that will provide a container with a given hostname and domain name.
* And many others... Please refer to the Wikipedia resource to learn more about Linux namespaces: 
https://en.wikipedia.org/wiki/Linux_namespaces

Containers will therefore be built using this set of available namespaces to isolate the necessary resources of the Linux Kernel.
Besides Linux namespaces you can also use cgroups to isolate the resource usage such as CPU or memory inside a container.

Control groups (cgroups) is another feature of the Linux Kernel initially released in 2007.
It allows for isolation of the resource usage of a set of processes.
You can control the usage of CPU and memory utilization, disk I/O throughput, network and many other resources.
It is not compulsory to set up your containers but it is highly recommend by Docker.
You can compromise otherwise the stability of your cluster.
