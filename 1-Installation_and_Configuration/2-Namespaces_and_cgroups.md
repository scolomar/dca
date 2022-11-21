## Section 1: Installation and Configuration

### Chapter 2: Namespaces and control groups

Namespaces and control groups make up the core of the underlying technology behind containers.

Linux namespaces are a feature of the Linux Kernel that was initially released in 2002.
They are the base for containers.
In fact they actually constitute what we know as containers.

Namespaces are partitions of the resources of the Linux Kernel so that a set of resources such as for example the process identifier (PID) is assigned to a specific Linux namespace and is only accessible from inside that namespace.
In this case PID namespaces are nested meaning that from the original namespace you will be able to view the PID of any other process running in a nested namespace but you will not view the PID of any other process running outside your own namespace.

There are different types of namespaces:

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
  * https://en.wikipedia.org/wiki/Linux_namespaces

Containers will therefore be built using this set of available namespaces to isolate the necessary resources of the Linux Kernel.
Besides using namespaces you can also use cgroups to isolate the resource usage such as CPU or memory inside a container.

Control groups (cgroups) is another feature of the Linux Kernel initially released in 2007.
It allows for isolation of the resource usage of a set of processes.
You can control the usage of CPU and memory utilization, disk I/O and network throughput, and many other resources.
It is not compulsory to set up your containers with control groups but it is officially considered as a best practice by Docker.
You can otherwise compromise the stability of your cluster in case your containers exhaust the hardware resources of the host machine.

It is important to understand the fundamental differences between containers and virtual machines.
A virtual machine is built on top of a set of virtual resources provided by a virtualization software that is the hypervisor.
The hypervisor will provide access to virtual resources built on top of the actual hardware of the host machine.
In the virtual machine we will install a full operating system including the kernel as well as our main application with all the necessary dependencies.
A typical example would be a virtual machine with Tomcat, MySQL or Apache web server.
Once our main application is running on the virtual machine it might need access to memory or disk.
For that purpose it will launch system calls to the guest operating system.
The hypervisor will then translate those into real system calls to access the actual physical hardware.
This way virtual machines are so secure because any system call is passed through the hypervisor before actually reaching the host machine.

Containers work in a completely different way.
They use namespaces and cgroups to access a partition of the resources of the host machine instead of using any kind of virtualization.
Containers have therefore absolutely nothing to do with virtualization despite some common confusion in this sense.
It is more about partitioning and isolation.
The security of a container will depend on how secure the technology behind namespaces and cgroups really is.
Despite the top level of security achievable by Linux namespaces it will never equate the security of a virtual machine provided by a bare-metal hypervisor.

Given that containers launch system calls directly to the host kernel there is nothing that prevents them from exhausting the actual physical resources and crash the host machine.
Namespaces ensure the isolation of some operating system resources such as filesystem mount points, hostname, network stack or PID but it does not limit the usage of memory, disk or CPU.
In order to control the usage of physical resources we need to configure the control groups or cgroups for that container.
I would highly recommend to set up cgroups for any container that is going to be used in a production environment.

## Exercise:
Open a terminal in any Linux system and run the following command:
```
pid=1
cat /proc/${pid}/cgroup
```
The output will be similar to the following:
```
11:perf_event:/
10:memory:/
9:freezer:/
8:hugetlb:/
7:devices:/
6:cpu,cpuacct:/
5:net_cls,net_prio:/
4:blkio:/
3:pids:/
2:cpuset:/
1:name=systemd:/
```
This result shows that process with PID 1 belongs to the control group / (root/global).
Any other non-containerized application running on the same machine will show a similar result. 
Like for example:
```
cat /proc/$$/cgroup
```
```
11:perf_event:/
10:memory:/
9:freezer:/
8:hugetlb:/
7:devices:/
6:cpu,cpuacct:/
5:net_cls,net_prio:/
4:blkio:/
3:pids:/
2:cpuset:/
1:name=systemd:/user.slice/user-1000.slice/session-4268.scope
```
These results basically mean that the Linux operating system by default separates the Kernel resources in control groups and namespaces. Each process running on the Linux system will be assigned a control group which can be checked under the location ```/proc/PID/cgroup```. This feature was available in the Linux Kernel many years before Docker was developed. This set of namespaces and control groups is what we call a "container". The container assigned by default to any process running on a Linux machine is sometimes called the global or root container. Docker uses this same technology in order to set up local Docker containers (nested namespaces of the global root namespace).

Let us see the difference between this global container and a local Docker container. For this purpose let us run the following command:
```
docker run --detach --entrypoint ping --name test --rm --tty library/busybox:latest localhost
```
Now we can check the process running inside this container and the control group assigned to it:
```
cat /proc/$( sudo docker top test | awk '!/PID/{ print $2 }' )/cgroup
```
The result shows the control group assigned to our process which coincides with Docker container ID:
```
11:perf_event:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
10:memory:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
9:freezer:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
8:hugetlb:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
7:devices:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
6:cpu,cpuacct:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
5:net_cls,net_prio:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
4:blkio:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
3:pids:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
2:cpuset:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
1:name=systemd:/docker/a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
```
We get the same ID when we check the Docker containers folder or use the Docker client command:
```
$ sudo ls /var/lib/docker/containers -l
total 0
drwx--x--- 4 root root 237 Apr  7 03:10 a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8
```
```
$ sudo docker ps --no-trunc
CONTAINER ID                                                       IMAGE                    COMMAND            CREATED         STATUS         PORTS     NAMES
a0870498d3b51140843b89f349e7d77ada7852f86fccf63cfa6169df083592f8   library/busybox:latest   "ping localhost"   4 minutes ago   Up 4 minutes             test
```
