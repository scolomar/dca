## Chapter 1: Installation and Configuration

### Section 3: Docker engine

The Docker engine is the core responsible for the building and managing of your containers.
It is open source and has a client-server architecture.

The server is the Docker daemon that will offer an API to any client request and perform the requested operation on the different Docker objects such as images, volumes, networks or containers.

The Docker daemon is written in the Go programming language (the same as Kubernetes) and uses the underlying Kernel features to provide its functionalities.
The main Kernel features used by the Docker daemon are: namespaces, control groups and UFS (Union File System).

The use of namespaces will permit the isolation of the containers at the networking, filesystem and processing level just to mention the most common.

The control groups will enhance the functionalities already provided by the namespaces allowing to limit or restrict the usage of resources inside a container such as memory and CPU.

The UnionFS will create a superposition of the different layers that constitute an image to provide a unified and consistent filesystem for the container. 
At this point I would like to remind you that the use of too many layers for our container image will obviously degrade the performance due to this UnionFS superposition.
In that sense the fewer the layers the better the performance of the container.

The Docker client is the other side of the coin.
It will connect to the Docker server and send the API requests to perform any operation such as building a Docker image, create a Docker network or run a Docker container.

The Docker client can be locally run in the same machine as the Docker server or executed in a remote machine (such as a bastion or the personal laptop of a developer).

The Docker engine is an extremely powerful tool that is easy to install and use.
It also provides an excellent functionality both in terms of stability and capacity.

To use Docker Swarm you do not need anything else than the core Docker engine. 
Without installing any other software you can enable the Swarm mode to provide orchestrating capabilities to your container deployment getting practically the same functionality as a Kubernetes cluster but consuming much less resources and definitely easier to configure.

## Exercise

Let us explore the different resources that have been created after a fresh default installation.

The first interesting location is `/var/lib/docker/` where you will find all resources created by Docker.
This folder is protected so that you need to be `root` user in order to explore its content.
It is important to understand that we can explore the content for learning or troubleshooting but we should never modify this content otherwise we might corrupt Docker installation.
Let us analyze the content of this folder and subfolders:
- `/var/lib/docker/containers/`: This folder contains all the created containers (running and stopped containers). 
As already said we should never modify the content of this folder which will be dynamically updated whenever we create or remove a container.
Each container will be stored in its own folder which name will exactly coincide with the full ID of the container.
The content will include files like: hostname, hosts, resolv.conf and even the logs of the container.
Any running  or exited container will have such a folder.
Once we remove the container the corresponding folder will be removed.
It is important to keep this folder available even for exited containers so as to troubleshoot what happened to that container.
Once removed it will be impossible to recover its content.
- Another interesting folder is: `/var/lib/docker/overlay2/`.
This folder contains the image layers of your local Docker images.
You can examine the content of any Docker image exploring this folder.
Remember that a Docker image is just a folder that contains libraries and dependencies necessary for your application to run inside the Docker container.
And that is what you will find in that folder: binaries, libraries, configuration files.
Similar to the root filesystem of any operating system but not necessarily.
It is custom to download Docker images that contain almost a full operating system but that is not actually needed.
The only necessary content is the needed libraries to run your application.
In order to run your application you will first need to compile an executable binary from the source code (or use an interpreted script).
When you compile a binary you do it static or dynamically linked.
If you statically compile an executable binary from the source code then no libraries will be needed to run the application.
In that case your Docker image can be completely empty (`FROM scratch`) because the process will run inside your container with all the necessary dependencies included in the executable.
But that is not normally possible.
It is very usual to dynamically compile your executable so that it will be linked to the necessary libraries in order to run the application.
In that case your Docker image should contain those libraries in the exact location that the binary is expecting them to be.
Many customers find it easier to use a base image which all Linux libraries to ease the running process of your containers. 
That is why it is typical to use Docker images that contain entries like: `FROM ubuntu`.
Nevertheless it will contain the Ubuntu libraries but never the Linux kernel.
The Docker container will always use the Linux kernel of the host operating system.
