## Chapter 3: Networking

### Section 2: Load balancing and DNS

One crucial aspect of Docker networking is how Docker handles the traffic towards the Docker services.
It is very important to understand this procedure in order to wisely develop secure and reliable applications.

Let us suppose we have a deployment of three container replicas serving our Tomcat application.
Each container will have a different IP obtained from a pool of available IPs.

According to a report published by DataDog the median average lifetime of a container is about 2.5 days meanwhile the lifetime of a virtual machine is almost 10x longer (23 days):
* https://imgix.datadoghq.com/img/docker-2017-8_v2.png

It is normal that containers fail.
There are many reasons why this happens.
Containers are limited in memory and CPU resources so that any issue with the running process may kill the container.
It is not the purpose of Docker or Kubernetes to prevent this situation.
Containers are not supposed to be reliable services.
On the contrary containers are supposed to be easily replaced whenever they fail.
This is the strategy followed by both Docker and Kubernetes: in case of failure just replace ASAP the failing container or Pod.
Therefore the correct strategy for containerized application development should be: if you have to fail then fail fast (so that you can be fast replaced).
There is no purpose in keeping a failing container alive or trying to restore its original state.
Containers are supposed to eventually fail and be fast replaced.
This is the principle that needs to guide our design.

What happens then when a container fails?
It will be substituted and most probably the previously assigned IP will be replaced by a new one freshly obtained from the pool of available IPs.
This can be a nightmare for the developer or the operator (or both) if they try to handle the traffic by themselves.
Instead of trying to control the traffic to the containers we need to trust the orchestrator.
Both Docker and Kubernetes have internal resources to transparently divert the traffic to the correct IP.
Docker has internal DNS resolution that will translate the name of the service to the actual IP.
So that the name of the service is for Docker a FQDN that will be internally translated without requiring any other effort or configuration from the developer.
This means that our source code should point to service names instead of pointing to real IPs.
Docker (or Kubernetes) will do the translation for us.

This is an example of how our software should be written when trying to connect to a containerized microservice.
This sample is the configuration of an Nginx reverse proxy:
```
server {
  listen 80;
  location / {
    index  index.html index.htm;
    root /usr/share/nginx/html;
  }
  location /target/ {
    proxy_pass http://target;
  }
  server_name localhost;
}
```

As you can see in the example the `proxy_pass` directive is forwarding the traffic to `target` that is the name of another Docker service waiting for connections as shown in the Docker compose file:
```
services:
  proxy:
    image: nginx
    deploy:
      replicas: 2
    ports:
    - "80"
  target:
    image: nginx
    deploy:
      replicas: 3
```

We have deployed a total of 5 replicas: 2 replicas for the reverse proxy and another 3 replicas for the target microservice.
In both cases Docker will internally handle the communication between the containers.
We do not need to specify any IP or even publish any port.
Everything is transparently handled by Docker.

Kubernetes works in a very similar way though the configuration of the Kubernetes compose file would be a bit more complex.
Kubernetes has the same feature as Docker to handle the service traffic and transparent DNS resolution of service names.
Docker is nevertheless monolithic meanwhile Kubernetes is modular.
This implies that in Docker you can configure all the details in a service block of the Docker compose file meanwhile in Kubernetes you need to separatedly configure several independent objects: at least two Deployments and two Services:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: proxy-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      microsvc: proxy-svc
  template:
    metadata:
      labels:
        microsvc: proxy-svc
    spec:
      containers:
      - name: proxy-cont
        image: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: target-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      microsvc: target-svc
  template:
    metadata:
      labels:
        microsvc: target-svc
    spec:
      containers:
      - name: target-cont
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: proxy
spec:
  ports:
  - port: 80
  selector:
    microsvc: proxy-svc
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: target
spec:
  ports:
  - port: 80
  selector:
    microsvc: target-svc
```

This is a nice example on how much easier is Docker in comparison to Kubernetes.
