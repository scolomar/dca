## Chapter 3: Networking

### Section 4: Ports and services

Docker services are by default internal.
When we create a service it will imply one or more containers running the same task.

For example a web service may consist of a hundred instances of Apache Web server with exactly the same configuration.
When there is a request for the web service it will be forwarded to any of the available instances that will process the request and answer it.
This is a great solution in order to ensure for example the availability of an online shop during Black Friday or a voting website during election period.

The service will hold a virtual IP and the traffic will be forwarded to the different instances of the service that will hold their own private IP.
You do not even need to know the virtual IP of the service or the private IP of the target container.
There will be native DNS resolution of the Docker service into its virtual IP if the container that wants to consume the service is attached to the same Docker network as the target.
You nevertheless need to point to the correct port when trying to query the service like for example http://myservice:8080.

The service will also act as a virtual load balancer that will regularly check the health of the target containers and discard those which are not ready.
The traffic will therefore only be forwarded to healthy container instances of the specified service.

By default the Docker service will only attend internal requests coming from another container in the same Docker network.
If we want to publish our Docker service so as to also serve external requests coming from an external network then we need to modify the default configuration.
We need to publish the port of the host machine that will be mapped to the port of the running container.
With this configuration any request coming from the external network to a specific port of the host machine will be forwarded to the corresponding port of the container if properly mapped and published.

For example an Apache Web container will be able to communicate with port 8080 of the Tomcat container only available through the internal Docker network.
At the same time we may publish port 80 of the host machine that will be mapped to port 80 of the Apache container so as to allow external access to the Apache web service.

It is in general recommended not to publish any port unless absolutely necessary.
Remember that we do not need to publish a port if we only want to consume that service from another container in the same network.

## Exercise

Let us deploy a basic web server:
```
docker run --detach --name webserver nginx:alpine
webserver_ip=$( docker inspect webserver | grep IPAddress.: -m 1 | cut -d '"' -f 4 )
```
This command will successfully deploy an Nginx web server but it will only be available to any other container connected to the same network.
By default all containers are connected to the default bridge network so that any container will be able to connect:
```
$ docker run --entrypoint curl nginx:alpine ${webserver_ip} --head --silent
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Mon, 13 Jun 2022 01:08:02 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
Connection: keep-alive
ETag: "61f01158-267"
Accept-Ranges: bytes
```
The connection is working perfectly fine but we need to use the IP address of the container since we are connected to the default network.
If we want to use custom domain names for the connection then we need to use custom networks:
```
docker network create my-network
docker run --detach --name webserver2 --network my-network nginx:alpine
```
Now we can connect to the web server using the name of the container instead of its IP address:
```
$ docker run --entrypoint curl --network my-network nginx:alpine webserver2 --head --silent                                      
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Mon, 13 Jun 2022 01:12:29 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
Connection: keep-alive
ETag: "61f01158-267"
Accept-Ranges: bytes
```
Let us go one step further and let us deploy a Docker service instead of a standalone container.
We will also need to create a network but we will create this time an overlay network instead of the default bridge network.
We need to use overlay networks when connecting Docker services:
```
docker network create --driver overlay my-overlay-network
docker service create --name webserver --network my-overlay-network nginx:alpine
```
By default it has created one single replica of our Nginx web server but we can scale it up:
```
docker service scale webserver=2
```
Now that we have 2 replicas of our Nginx web server we can test the connection.
For that purpose we will create another service to connect.
The reason to create a service instead of a single container is that we cannot connect any container to an overlay network.
Only service can connect to overlay networks.
This service will create a container that will test the connection and after completion it will exit without restart.
We will be able to check the successful connection exploring the logs of this exited container:
```
docker service create --detach --entrypoint curl --name test --network my-overlay-network --restart-condition none nginx:alpine webserver --head --silent
```
Now we can check the logs and verify that the connection was successful:
```
$ docker service logs test
test.1.zkhfm3jiiq3c@node1    | HTTP/1.1 200 OK
test.1.zkhfm3jiiq3c@node1    | Server: nginx/1.21.6
test.1.zkhfm3jiiq3c@node1    | Date: Mon, 13 Jun 2022 01:28:09 GMT
test.1.zkhfm3jiiq3c@node1    | Content-Type: text/html
test.1.zkhfm3jiiq3c@node1    | Content-Length: 615
test.1.zkhfm3jiiq3c@node1    | Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
test.1.zkhfm3jiiq3c@node1    | Connection: keep-alive
test.1.zkhfm3jiiq3c@node1    | ETag: "61f01158-267"
test.1.zkhfm3jiiq3c@node1    | Accept-Ranges: bytes
test.1.zkhfm3jiiq3c@node1    | 
```
We have directly connected to the service but we could also have connected using the IP address of the container.
```
replica_ip=$( docker inspect $( docker ps --quiet | tail -1 ) | grep IPAddress.: | tail -1 | cut -d '"' -f 4 )
docker service create --detach --entrypoint curl --name test-ip --network my-overlay-network --restart-condition none nginx:alpine ${replica_ip} --head --silent
```
Now we can check the logs of the test container and verify its success:
```
$ docker service logs test-ip
test-ip.1.o8ipt7ts4dzc@node1    | HTTP/1.1 200 OK
test-ip.1.o8ipt7ts4dzc@node1    | Server: nginx/1.21.6
test-ip.1.o8ipt7ts4dzc@node1    | Date: Mon, 13 Jun 2022 01:43:10 GMT
test-ip.1.o8ipt7ts4dzc@node1    | Content-Type: text/html
test-ip.1.o8ipt7ts4dzc@node1    | Content-Length: 615
test-ip.1.o8ipt7ts4dzc@node1    | Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
test-ip.1.o8ipt7ts4dzc@node1    | Connection: keep-alive
test-ip.1.o8ipt7ts4dzc@node1    | ETag: "61f01158-267"
test-ip.1.o8ipt7ts4dzc@node1    | Accept-Ranges: bytes
test-ip.1.o8ipt7ts4dzc@node1    | 
```
