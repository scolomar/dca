## Section 1: Installation and Configuration

### Chapter 2: Namespaces and control groups

Namespaces and control groups make up the core of the underlying technology behind containers.

Linux namespaces are a feature of the Linux Kernel that was initially released in 2002. They are the foundation of what we know as containers.

Namespaces are partitions of Linux kernel resources such that a set of resources, such as the process identifier (PID), is assigned to a specific Linux namespace and can only be accessed from within that namespace. In this case, the PID namespaces are nested, which means that from the root namespace you will be able to see the PID of any other process running in a nested namespace but you won't see the PID of any other process running outside of its own namespace.

There are different types of namespaces. Let's review the most used:
* PID: For the process identifier (PID). This will isolate the PID of processes running in different namespaces. You will not have visibility into the PID of any processes running outside of your container.
* MNT: For file system mount points. This namespace will allow the isolation of the file system mounted within a container from any other neighboring containers.
* NET: For the network stacks. This namespace will provide visibility into the networking stack for a specific container and will fully isolate that networking stack from any other container. This will also allow the IP range to overlap within different containers. The routing table will also be private to that container or network namespace. Neutron uses this same technology to provide network isolation for the Openstack platform.
* IPC: For Inter Process Communications. Processes running inside a container can safely use the host machine's shared memory resources, and those resources will be private to the container or IPC namespace.
* UTS: For Unix timesharing namespaces that will provide a container with a given hostname and domain name.
* And many others... Please refer to the Wikipedia resource to learn more about Linux namespaces: 
  * https://en.wikipedia.org/wiki/Linux_namespaces

Therefore, containers will be built using this set of available namespaces to isolate the necessary Linux Kernel resources. In addition to using namespaces, you can also use control groups to isolate the usage of resources like CPU or memory within a container.

Control groups (cgroups) are another feature of the Linux kernel initially released in 2007. It allows for the isolation of resource usage by a set of processes. You can control CPU and memory usage, disk I/O and network performance, and many other resources. It is not mandatory to configure your containers with control groups, but Docker officially considers it a best practice. Otherwise, you may compromise the stability of your cluster in case your containers exhaust the hardware resources of the host machine.

### Containers vs virtual machines

It is also important to understand the fundamental difference between containers and virtual machines. A virtual machine is built on a set of virtual resources provided by virtualization software, which is the hypervisor. The hypervisor will provide access to the virtual resources created on top of the actual hardware of the host machine. In the virtual machine we will install a complete operating system including the kernel as well as our main application with all the necessary dependencies. A typical example would be a virtual machine with a full operating system plus a Tomcat, MySQL, or Apache web server.

Once our main application is running in the virtual machine, it may need access to memory or disk. It does this by launching system calls to the guest operating system. The hypervisor will then translate these into actual system calls to access the actual physical hardware. In this way, virtual machines are so secure because any system calls go through the hypervisor before reaching the host machine.

Containers work in a completely different way. They use namespaces and control groups to access a partition of the host machine's resources instead of using any kind of virtualization. Therefore, containers have absolutely nothing to do with virtualization despite some common confusion in this regard. It's more about partitioning and isolation.

The security of a container will depend on how secure the technology behind the namespaces and control groups actually is. Despite the highest level of security that Linux namespaces can achieve, it will never match the security of a virtual machine provided by a bare-metal hypervisor.

Since containers launch system calls directly to the host kernel, there is nothing to stop them from exhausting actual physical resources and crashing the host machine.

Namespaces ensure the isolation of some operating system resources, such as file system mount points, hostname, network stack, or PID. But they do not limit the use of memory, disk or CPU. To control the use of physical resources, we need to configure the control groups for that container. It is strongly recommended to configure control groups for any container that will be used in a production environment.

## Exercise: Containers, namespaces and cgroups

Open a terminal on any Linux system and run the following command:
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
This output shows that the process with PID 1 belongs to the root (global) control group. Any other non-containerized applications running on the same machine will show similar output. For example:
```
cat /proc/$$/cgroup

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
These results basically mean that the Linux operating system by default separates kernel resources into control groups and namespaces. Every process running on the Linux system will be assigned a control group that can be checked at the location `/proc/$PID/cgroup`.

This feature was available in the Linux kernel many years before Docker was developed. This set of namespaces and control groups is what we call a "container". The container assigned by default to any process running on a Linux machine is also called the global or root container. Docker uses this same technology to configure local Docker containers (typically nested namespaces of the global root namespace).

Let's see the difference between this global container and a local Docker container. To do this, we execute the following command:
```
docker run --detach --name test busybox ping localhost
```
We can now check the process running inside this container and the control group assigned to it:
```
cat /proc/$( sudo docker top test | awk '!/PID/{ print $2 }' )/cgroup
```
The output shows the control group assigned to our process that matches the Docker container ID:
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
