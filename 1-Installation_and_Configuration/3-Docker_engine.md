## Section 1: Installation and Configuration

### Chapter 3: Docker engine

The Docker engine is the core responsible for creating and managing your containers. It is open source and has a client-server architecture.

The server is the Docker daemon that will offer its API to any client request and perform the requested operation on the different Docker objects, such as images, volumes, networks or containers.

The Docker daemon is written in the Go programming language (same as Kubernetes) and uses the underlying features of the Kernel to provide its functionality. The main kernel features used by the Docker daemon are: namespaces, control groups, and UFS (Union File System).

The use of namespaces will allow the isolation of containers at the network, file system and processing levels, to mention the most common.

Control groups will enhance functionality already provided by namespaces, allowing you to limit or restrict the use of resources within a container, such as memory and CPU.

UnionFS will create an overlay of the different layers that make up an image to provide a unified and consistent file system for the container. At this point, it's important to note that using too many layers for the container image will obviously degrade performance due to this UnionFS overlap. In that sense, the fewer layers there are, the better the performance of the container.

The Docker client is the other side of the coin. It will connect to the Docker server and send API requests to perform any operation, such as building a Docker image, building a Docker network, or running a Docker container.

The Docker client can run locally on the same machine as the Docker server or run on a remote machine (such as a bastion or a developer's personal laptop).

The Docker engine is an extremely powerful tool that is easy to install and use. It also provides excellent functionality both in terms of stability and capacity.

To use Docker Swarm you don't need anything other than the core Docker engine. Without installing any other software, you can enable Swarm mode to provide orchestration capabilities to your container deployment, getting pretty much the same functionality as a Kubernetes cluster but consuming far fewer resources and definitely easier to set up.

## Exercise: Exploring the Docker engine

Let's explore some resources that are created after a fresh installation of the Docker engine.

### /var/lib/docker/
In this interesting location you will find all the resources created by Docker. This folder is protected, so you must be the root user to be able to browse its contents. It's important to understand that we can explore the content to learn or troubleshoot, but we should never modify this content, otherwise we might break the Docker installation.

Since all Docker objects are stored locally in /var/lib/docker/, it's a good idea to mount this folder as a separate partition for ease of maintenance and to ensure data persistence and reliability.

Let's analyze the contents of this folder and subfolders:
* /var/lib/docker/containers/
* /var/lib/docker/overlay2/
* /var/lib/docker/volumes/

### /var/lib/docker/containers/
This folder contains all created containers (running and stopped containers). As already said, we should never modify the content of this folder, which will dynamically update each time we create or delete a container. Each container will be stored in its own folder whose name will exactly match the full container ID. The content will include files such as: hostname, hosts, resolution settings, and even the container logs.

Any running or closed container will have such a folder. Once we delete the container, the corresponding folder will be deleted. It is important to keep this folder available even for closed containers to fix any issues that happened to that container. Once deleted it will be impossible to recover its content.

### /var/lib/docker/overlay2/
This folder contains the image layers of your local Docker images. You can browse the contents of any Docker image by browsing this folder. Remember that a Docker image is just a folder that contains libraries and dependencies needed for your application to run inside the Docker container. And that's what you'll find in that folder: binaries, libraries, configuration files. Similar to the root file system of any operating system but not necessarily.

It's customary to download Docker images that contain almost a complete operating system, but you don't really need it. The only content required is the libraries required to run your application. To run your application, you'll first need to compile an executable binary from source (or use an interpreted script).

When you compile a binary you make it static or dynamically linked. If you statically compile an executable binary from source, no libraries are needed to run the application. In that case, your docker image can be completely empty (FROM scratch) because the process will run inside your container with all necessary dependencies included in the executable.

But normally it is not possible to statically compile the code. It is very common to dynamically compile the executable so that it links with the libraries needed to run the application. In that case, the Docker image should contain those libraries in the exact location where the binary expects them.

Many customers find it easier to use a base image that contains all the commonly used Linux libraries to make image and container management easier. That's why you'll see Dockerfiles with an entry like this:
```
FROM ubuntu
```
This image will contain all the Ubuntu libraries but never the Linux kernel. The Docker container will always use the kernel of the host operating system.

Any changes to the container's file system will be visible in the corresponding subfolder. For any container, you will find a diff subfolder that will contain any changes to its filesystem. For example, if you create a new file (or modify an old file), that new file (or the new version of the old file) will be visible in the diff subfolder.

### /var/lib/docker/volumes/
This folder will contain the Docker volumes that have been created. Any content from these volumes will be found in the corresponding subfolders. Any changes to the content of these subfolders will automatically be reflected in the volume mounted in the running container.


In addition to these interesting locations on the host file system, there are other interesting resources that have been created by default, such as networks. By default, Docker will create three networks:
* Bridge
* Host
* None

### Bridge network
This network will be used to connect (by default) any container created. This means that all containers will be able to communicate over this network. It is actually recommended to create custom networks for each deployment, so that each custom network is isolated from each other. Only containers on the same network will be able to communicate with each other.

### Host network
This is the network of the host operating system. If we create a container attached to this network, it will have access to the same network resources as any other (non-containerized) process running on the host.

### None network
Any container connected to this network will only have the loopback network interface (localhost) available. This can be useful when we want to completely isolate our containers from any network for security reasons (such as a root certificate authority).
