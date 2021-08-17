## Chapter 6: Security

### Section 3: Network security

Docker networks are secure by default.

Docker networks are fully isolated by default. Therefore we cannot communicate two running containers unless they are attached to the same network.
One container can be attached to as many networks as needed so that communication and security are both guaranteed when using Docker networks.

In a three tier architecture we can place for example a frontend container attached to the frontend network and a middleware container attached to both the frontend and the backend network.
The database container will only be attached to the backend network: this way the middleware will be able to talk to both the frontend and the database because it will be attached to both networks.
The frontend will only be able to talk to the middleware but not to the database because they will be in different networks. 
An example of such an architecture could be an Apache Web server as a frontend, a Tomcat server as a middleware and a MySQL server as a database.

A different situation arises when using Kubernetes. 
The Kubernetes network is flat by definition. Therefore Kubernetes networks are not secure by default.
We need to ensure the security of the Kubernetes flat network by applying Network Policies. 
Without the use of a Network Policy any container running inside a Pod can talk by default to any other container running inside another Pod if they are both in the same Kubernetes cluster.
