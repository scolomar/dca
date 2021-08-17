## Chapter 3: Networking

### Section 3: Ports and services

Docker services are by default internal.
When we create a service it will imply one or more containers running the same task.

For example a web service may consist of a hundred instances of Apache Web server with exactly the same configuration.
When there is a request for the web service it will be forwarded to any of the available instances that will process the request and answer it.
This is a great solution in order to ensure for example the availability of an online shop during Black Friday or a voting website during election period.

The service will hold a virtual IP and the traffic will be forwarded to the different instances of the service that will hold their own private IP.
You do not even need to know the virtual IP of the service or the private IP of the target container.
There will be native DNS resolution of the Docker service into its virtual IP if the container that wants to consume the service is attached to the same Docker network as the target.
You nevertheless need to point to the correct port when you try to query the service like for example http://myservice:8080.

The service will also act as a virtual load balancer that will regularly check the health of the target containers and discard those which are not ready.
The traffic will therefore only be forwarded to healthy container instances of the specified service.

By default the Docker service will only attend internal requests coming from another container in the same Docker network.
If we want to publish our Docker service so as to also serve external requests coming from a external network then we need to modify the default configuration.
We need to publish the port of the host machine that will be mapped to the port of the running container.
With this configuration any request coming from the external network to a port of the host machine will be forwarded to the corresponding port of the container if properly mapped and published.

For example an Apache Web container will be able to communicate with port 8080 of the Tomcat container only available through the internal Docker network.
At the same time we may publish port 80 of the host machine that will be mapped to port 80 of the Apache container so as to allow external access to the Apache web service.

It is in general recommended not to publish any port unless absolutely necessary.
Remember that we do not need to publish a port if we only want to consume that service from another container in the same network.
