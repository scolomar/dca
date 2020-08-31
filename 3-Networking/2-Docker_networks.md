## Chapter 3: Networking

### Section 2: Docker networks

One of the main advantages of using Docker is the "batteries included" networking.
Docker networking is more efficient and secure than Kubernetes networking and does not need to install any additional plugin: Docker networking is secure by default.
Docker uses Linux bridges to build the default Docker networks.
Docker uses virtual ethernet devices to connect different networks.
One end of the veth pair is placed in one network and the other end is placed in another network.
Docker creates iptables rules to prevent unwanted communication between different Docker networks.
By default any Docker container cannot talk to any other container that is attached to a different network.
Only containers attached to the same network can talk to each other.
We achieve this way a high level of security by default.

The Linux bridge is not the only type of network available in Docker but it is the default type when creating a Docker network in a host machine.
To connect containers from different remote host machines then we need another type of network: the overlay network.

FROM 5 TO 6
