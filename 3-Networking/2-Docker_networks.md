## Chapter 3: Networking

### Section 2: Docker networks

One of the main advantages of using Docker is the "batteries included" networking.
Docker networking is very efficient and secure and does not need to install any additional plugin.
Docker networking is secure by default.

Docker uses Linux bridges to build the default Docker networks and virtual ethernet devices to connect different networks.
One end of the veth pair is placed in one network and the other end is placed in another network.
The Linux network namespaces guarantee the isolation of the different network stacks.
Docker also creates iptables rules to prevent unwanted communication between different Docker networks.
By default any Docker container will not be able to talk to any other container that is attached to a different network.
Only containers attached to the same network can talk to each other.
Docker achieves this way a high level of security by default.

The Linux bridge is not the only type of network available in Docker but it is the default type when creating a Docker network in a host machine.
Let us suppose a simple 3-tierd example with an Apache web server frontend, a Tomcat backend and a MySQL database all deployed in a standalone host machine.
For the application deployment let us create three different bridge networks: frontend, backend and database.
Docker will create a separate Linux bridge for each network.
By default there will be no connectivity through the three different networks because of the iptables isolation rules.

If we want for example the Tomcat backend to be able to talk to the MySQL database then we need to attach both containers to the same network otherwise the communication will not be possible.
We do not need to configure anything else.
By default any two containers attached to the same network can freely talk to each other.
There is no need to "publish" that port as a service.
When we publish a service with the `--publish` option we mean to connect the incoming traffic from the public network interface of the host machine to the private network of the target container.
This is only necessary when we want to provide access to our service from an external public network like for example to allow an external client to connect to our Tomcat or Apache frontend.
There is no need to publish any service when we only want to allow internal communication between services in the same deployment.
For this purpose it is enough with attaching both services to the same Docker network.

This last example was of an application deployment on a standalone machine. 
But what about a deployment on a cluster of several remote host machines?
How can we communicate containers located in different host machines?
For that purpose we need another type of network that will connect the remote Docker hosts: the overlay network.
The virtual overlay network will use the physical underlay network to connect the services running in a group of remote host machines.
This overlay network will allow communication from host to host and the corresponding local bridge network will handle the traffic inside the host machine to reach the target container.

Overlay networks are created automatically when deploying an application with Docker Swarm but they can also be created manually with the command `docker network create`.
They are absolutely necessary when we need to communicate services that are running in different host machines.

#### Encryption

When using Docker Swarm all the management and control traffic will go through the overlay network to the remote Docker daemons and encrypted by default.
The application data traffic will nevertheless not be encrypted by default.
We can also configure the data traffic to be encrypted.
For that purpose we will specify that option when creating the overlay network.
That will enable IPSEC encryption at the VXLAN level.
This encryption will nevertheless impose a performance penalty and should be tested before being implemented in a production environment.

## Exercise

A possible application use case could be like this:
- A frontend container running an Nginx web server would only be connected to the frontend network.
- A backend container running Tomcat would be connected both to the frontend network and to the backend network.
- A database container running MySQL would only be connected to the backend network.

In this example scenario only the Tomcat container would therefore have access to the MySQL database network. 
The Nginx web server could only talk to the Tomcat container but not to the Database.

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
  frontend-network:
services:
  backend:
    image: tomcat:alpine
    networks:
      - backend-network
      - frontend-network
  database:
    environment:
      MYSQL_ROOT_PASSWORD: xxx
    image: mysql
    networks:
      - backend-network
  frontend:
    image: nginx:alpine
    networks:
      - frontend-network
    ports:
      - 80
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
We can again check the connectivity between containers with the following commands:
```
docker exec TEST_frontend.1.rkwqacwte7shbl8hqedrb2j51 ping backend
docker exec TEST_backend.1.uxsl2etegsh4o5pskjxq96bhk ping frontend
docker exec TEST_backend.1.uxsl2etegsh4o5pskjxq96bhk ping database
```
Most probably you have noticed the weird name for the container when using Docker stack. Please find below the full syntax for the container name:
- [Stack name]_[Service name].[Task number].[Task ID]
