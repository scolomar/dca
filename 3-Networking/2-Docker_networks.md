## Chapter 3: Networking

### Section 2: Docker networks

One of the main advantages of using Docker is the "batteries included" networking.
Docker networking is more efficient and secure than Kubernetes networking and does not need to install any additional plugin: Docker networking is secure by default.

Docker uses Linux bridges to build the default Docker networks and virtual ethernet devices to connect different networks.
One end of the veth pair is placed in one network and the other end is placed in another network.
The Linux network namespaces guarantee the isolation of the different network stacks.
Docker also creates iptables rules to prevent unwanted communication between different Docker networks.
By default any Docker container will not be able to talk to any other container that is attached to a different network.
Only containers attached to the same network can talk to each other.
Docker achieves this way a high level of security by default.

The Linux bridge is not the only type of network available in Docker but it is the default type when creating a Docker network in a host machine.
Let us suppose a simple 3-tierd example with a Tomcat frontend, a NodeJS-based API backend and a MySQL database all deployed in a standalone host machine.
For the application deployment let us create three different bridge networks: frontend, backend and database.
Docker will create a separate Linux bridge for each network.
By default there will be no connectivity through the three different networks because of the iptables isolation rules.

If we want for example the Tomcat frontend to be able to talk the MySQL database then we need to attach both containers to the same network otherwise the communication will not be possible.
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
