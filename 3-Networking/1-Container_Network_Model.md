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
docker run --detach --name frontend-container --network frontend-network nginx:alpine
```
In this case we have created a dummy container running an Nginx web server.
We would need to properly configure this web server in order to get a real application up and running.
The only purpose of this container is to show the network connectivity.
The `frontend` network is still isolated.
If we want to share or map the container network to the host network then we will need to run the container with the `publish` option:
```
docker run --detach --name frontend-container --network frontend-network --publish 80 nginx:alpine
```

Let us run another dummy container pretending to be a database running on the backend network:
```
docker run --detach --env MYSQL_ROOT_PASSWORD=xxx --name database-container --network backend-network mysql
```

If we want to communicate our backend application with the frontend and also with the database the we need to connect it to both networks.
That is not possible with the Docker CLI.
You can only create a new container attached to a single network when using the Docker command line.
For that purpose I will first create the container attached to the backend network and later I will connect this same container to the frontend network:
```
docker run --detach --name backend-container --network backend-network tomcat:alpine
docker network connect frontend-network backend-container
```

We can test the connectivity between containers with the following commands:
```
docker exec frontend-container ping backend-container 
docker exec backend-container ping frontend-container
docker exec backend-container ping database-container
```

With the command `docker inspect` we can check which containers are attached to each network:
```
$ docker inspect backend-network | grep Containers -A15
        "Containers": {
            "29a13c6eed6547720bdbfbb470d504efb4b127b121e47c184456b0aa30f97098": {
                "Name": "database-container",
                "EndpointID": "24abc67ad9f41c261cec9a693beb88c8f50467160fa0a783faa84424bb0b7a43",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            },
            "85161ab6b41b6889f39577833c6816742e944a8f4ae3b4442c4f9419ead1e57a": {
                "Name": "backend-container",
                "EndpointID": "21191d2c167fa3da08e467f14493ecc7c733096f33e1821683a0ad5840e802a2",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            }
        },
```
As we can see from the previous output both containers backend and database are connected to the backend network.
From the following command we can see that both backend and frontend containers are connected to the frontend network.
```
$ docker inspect frontend-network | grep Containers -A15
        "Containers": {
            "85161ab6b41b6889f39577833c6816742e944a8f4ae3b4442c4f9419ead1e57a": {
                "Name": "backend-container",
                "EndpointID": "58e24df3cb58869428caab0f315e39c3c23ade3a8fb6b9200d2a5d57d4cea3c2",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "d9bfba5fc5f0bd681b6b7b99f3b13ce2d28829177d18bb4a6b4cdb0ec7cafe89": {
                "Name": "frontend-container",
                "EndpointID": "a4740b2890537f348e0cbafe1328bc49ae8140850a2868780655cdd7e6ac0877",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
```

The `docker run` command has the limitation of one single network when creating a new container.
If we want to attach a second (or third) network then we need to use the command `docker network connect`.
Let us see how we can use a Docker compose file to configure both networks at the same time:
```
networks:
  backend-network:
    internal: true
  frontend-network:
    external: false
services:
  backend:
    image: busybox
    networks:
      - backend-network
  database:
    image: busybox
    networks:
      - backend-network
  frontend:
    image: busybox
    networks:
      - frontend-network
    ports:
      - 8080
version: '3.8'
```
Let us save the content in a file named `docker-compose.yaml` and deploy the stack with the following command:
```
docker stack deploy --compose-file docker-compose.yaml TEST
```
This is the output of the command:
```
Creating network TEST_backend-network
Creating network TEST_frontend-network
Creating service TEST_backend
Creating service TEST_database
Creating service TEST_frontend
```
It will first create the networks with the name of the stack `TEST_` as a prefix.
Once the two networks have been created it will create the three services: `backend`, `database` and `frontend` (also with the same prefix `TEST_`).
We can again use `docker inspect` command to check that containers are attached to their proper networks.
