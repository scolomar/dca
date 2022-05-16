## Chapter 3: Networking

### Section 1: Container Network Model

The Container Network Model (CNM) is an abstraction of the networking for Docker containers implemented by libnetwork.
The CNM is built on the following three main components: 
- Endpoint
- Network
- Sandbox

#### Endpoint
An Endpoint connects a Sandbox to a Network.
An example of its implementation could be a veth pair in Linux, or an Open vSwitch internal port.
An Endpoint can only belong to a single Network and to a single Sandbox (if connected).
Any Sandbox can nevertheless be connected to many endpoints.
Any Network can also be connected to many different endpoints.
An Endpoint basically represents the connection of a Sandbox (container) to a Network.
One Sandbox (container) can therefore be connected to many Networks through many different Endpoints (virtual Ethernet pairs).

#### Network
A Network is a group of different Endpoints which can directly communicate to each other.
An example of its implementation could be a Linux bridge, or an Open vSwitch.

#### Sandbox
A Sandbox represents the configuration of a container's network stack.
This includes the container's network interfaces, the routing tables and DNS entries.
An example of its implementation could be a Linux Network Namespace.

#### Summary
In order to summarize let us say that different Sandboxes (containers) can have many Endpoints (virtual Ethernet pairs) to connect to many different Networks.
Two different Sandboxes (containers) need to be connected through Endpoints (virtual Ethernet pairs) to the same Network if we want to communicate them.
Any two different Sandboxes (containers) which are not connected to the same Network cannot talk to each other.

The Container Network Model is as simple as this: in order to enable communication between containers we need to connect them to the same network otherwise the communication is not possible. Any container can be connected to many different networks.

## Exercise

A possible application use case could be like this:
- A frontend container running an Apache web server would only be connected to the frontend network.
- A backend container running Tomcat would be connected both to the frontend network and to the backend network.
- A database container running PostgreSQL would only be connected to the backend network.

In this example scenario only the Tomcat container would therefore have access to the PostgreSQL database network. 
The Apache web server could only talk to the Tomcat container but not to the Database.

Let us create such a configuration with Docker CLI.
The following commands will create the two networks: frontend and backend.
```
docker network create frontend-network
docker network create backend-network
```
Let us create a frontend container that will only be connected to the frontend network:
```
docker run --detach --name frontend-container --network frontend-network --tty busybox
```
In this case we have created a dummy container that is doing nothing but running a Linux shell.
The only purpose of this container is to show the network connectivity.
The `frontend` network is still isolated.
If we want to share or map the container network to the host network then we will need to run the container with the `publish` option:
```
docker run --detach --name frontend-container --network frontend-network --publish 8080 --tty busybox
```
Again the container is not running any application listening on port 8080 as this is just a proof of concept.

Let us run another dummy container pretending to be a database running on the backend network:
```
docker run --detach --name database-container --network backend-network --tty busybox
```

If we want to communicate our backend application with the frontend and also with the database the we need to connect it to both networks.
That is not possible with the Docker CLI.
You can only create a new container attached to a single network when using the Docker command line.
For that purpose I will first create the container attached to the backend network and later I will connect this same container to the frontend network:
```
docker run --detach --name backend-container --network backend-network --tty busybox
docker network connect frontend-network backend-container
```

This limitation is exclusive of the `docker run` command.
If we choose to use a Docker compose file then both networks can be perfectly configured and created using `docker stack deploy`:
```
networks:
  backend-network
  frontend-network
services:
  frontend:
    image: busybox
    network:
      - frontend-network
version: '3.8'
```
