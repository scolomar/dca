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

