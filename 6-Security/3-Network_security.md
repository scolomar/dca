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

## Exercise

The following command will show the networks that exist by default in Docker:
```
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
67e8976f4c41   bridge    bridge    local
9104adc6288e   host      host      local
feaae3674b52   none      null      local
```

Let us create two containers without specifying any network:
```
docker run --detach --name test1 --tty busybox
docker run --detach --name test2 --tty busybox
```

By default these two containers will be attached to the default network bridge as can be checked by running the following command:
```
$ docker inspect bridge | grep Containers -A15
        "Containers": {
            "7c3a2eceacdd0e60b744d2e536b241aec03b1e66d38779a6823d4370158c9d62": {
                "Name": "test2",
                "EndpointID": "0c0beaf7153fadd14b9506906cb716303d8f18f932d50cb335507d0b51923b25",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "cb38e941255e56b8c72611dc0ee8993b4ad456b1d22da7386f46b130c1940887": {
                "Name": "test1",
                "EndpointID": "e27c8057131a32cefe3588832ebc330e78aa5ec2851608fe1fda4ddd2f94188b",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
```

As they are connected to same network we can establish a communication between them but using the name of the container instead of its IP address will fail:
```
$ docker exec test1 ping test2
ping: bad address 'test2'
```

The communication will work if we specify the IP address of the target container:
```
$ docker exec test1 ping 172.17.0.3 -c 3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.158 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.088 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.132 ms

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.088/0.126/0.158 ms
```

If we want to use names instead of IP addresses we need to connect the containers to a custom network bridge:
```
docker network create my_bridge
docker run --detach --name test3 --network my_bridge --tty busybox
docker run --detach --name test4 --network my_bridge --tty busybox
```
Now the communication will be possible using just the name of the container:
```
$ docker exec test3 ping test4 -c 3
PING test4 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.128 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.082 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.081 ms

--- test4 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.081/0.097/0.128 ms
```
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
