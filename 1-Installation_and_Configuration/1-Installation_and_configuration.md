## Chapter 1: Installation and Configuration

### Section 1: Installation and configuration

Docker is quite easy to install and configure. The most important thing to note is that a default Docker installation is already secure "by default". This is very important because that is not the case with Kubernetes.

If you are not careful when installing Kubernetes, you will end up with a container platform that is potentially hackable. In any case, securing Kubernetes is not a trivial task.

The default Docker network is secure enough for any standard production deployment. At the same time, Docker is fully configurable with different plugins and configuration files to customize network, storage, or any other critical factor.

You can install the Docker engine on Windows and Linux platforms. Since the container is using the Kernel of the host operating system, if we want to run a Windows container, we will need to install the Docker engine on a Windows server. If we install the Docker engine on a Linux host machine, any containers running there will use the Linux Kernel of the host operating system. Therefore, we will be able to exclusively run Linux containers on a Linux host machine.

The approach that many customers use is to have all three manager nodes run Docker on a Linux machine and have two different groups of worker nodes: a Linux group and a Windows group. This way, they will be able to deploy Windows and Linux applications on their container platform.

It is impossible to run a Windows container on a Linux worker.
But it is though possible to run a Linux container on a Windows server.
The reason is that Windows server has a Linux Kernel embedded in the Windows operating system allowing this way to run both type of containers on a Windows machine: Linux and Windows containers.
There are some features that are only availabel when running Linux containers on a Linux host machine.
One example of such a feature is the possibility to mount a volume in memory on a tmpfs filesystem.

The management of the filesystem and how Docker works under the hood is also different in the case of Windows and Linux.
It is true that Windows is providing with the Compute Service an equivalent set of features to mimic the Linux features concerning filesystems, control groups, namespaces and other Linux operating system capabilities but this Compute Service will never be exactly the same as its Linux counterpart because there are some Linux features that are just impossible to reproduce in a Windows operating system.

## Exercise:

In order to install Docker engine on your machine you need to follow this documentation:
- https://docs.docker.com/engine/install/

It can be something as easy as running ```sudo apt-get install docker.io``` though the exact process will depend on the Docker version you want to download and the operating system that is actually running on your machine.
