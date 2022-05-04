## Chapter 5: Orchestration

### Section 1: Docker Swarm

Docker Swarm was released approximately two years later than Kubernetes.
Kubernetes is a very well known orchestrator that can handle any workload of containers across many worker nodes.
Kubernetes is nevertheless very complex to configure and manage.

Docker Swarm is also an orchestrator and as such it can also handle the lifecycle of any workload of containers across many worker nodes.
Docker Swarm is nevertheless much easier to configure and manage than Kubernetes.
It is also faster to react to any changes in the workload as well as more stable and lightweight.

We can think of Docker Swarm as a light, monolithic version of Kubernetes.
When deploying Docker Swarm instead of Kubernetes you might need less hardware resources such as memory or CPU to handle the same amount of workload.

There are some specific cases where it is necessary to deploy Kubernetes instead.
You will need Kubernetes if you want to deploy a workload of containers that need access to hardware devices of the host machine such as for example video cameras.
Docker Swarm cannot handle such a workload because it does not permit privileged containers.

I personally use Docker Swarm whenever possible and Kubernetes only when absolutely necessary.

You do not need to install anything to use Docker Swarm since it is fully integrated in the Docker engine.
You only need to initialize the swarm with a very simple command: `docker swarm init`.
That is the only thing you need to do to start using Docker Swarm.
After initializing the swarm the Docker engine will be aware of the nodes that compose the cluster and distribute and manage the workload across all the worker nodes.

By default Docker will also distribute the workload through the manager nodes but it is a good practice to block this behaviour. 
Only when you are working in your own laptop could it be interesting to send workload to the manager. 
It is generally advisable to send the workload exclusively to the workers.

The nodes in your infrastructure are composed of physical or virtual machines with Docker Swarm initialized.

There are two types of nodes: managers and workers.
There is at least one manager that is called the leader.
The rest of the nodes can be promoted to managers or demoted to workers at any moment with the command `docker node promote` and `docker node demote` respectively.

The managers are responsible for the management of the cluster meanwhile the workers handle the workload that is the containers serving your business application.
It is recommended not to run your apps in the manager nodes because any failure or bug could jeopardize the stability of the cluster.
Kubernetes prohibits that possibility by default.
Docker Swarm will deploy the workload in the managers by default so that it is a good practice to configure your deployment in production so as to disable this option and deploy exclusive in the worker nodes.
This is very easily achieved with the option `--constraint node.role==worker` that should be configured in any production deployment without exception.

The worker nodes will therefore run the containers serving your applications meanwhile the manager nodes will control the scheduling of the workload as well as the status of all the objects in the swarm.

## Exercise

Let us do a very simple example on how you can install a production Swarm cluster in less than 5 minutes.

For this exercise you will need a (free) Docker account. You can get such an account on this link:
- https://www.docker.com/

Now it is time to go the Playground environment:
- https://labs.play-with-docker.com/

You will need to login with your previously created Docker account and then click "Start".

You will be presented with a blank dashboard that will expire in 3 hours. We will only need 3 minutes of that time to create our cluster.

First you need to add instances so click 5 times on the button "ADD NEW INSTANCE". This will create 5 instances in our new lab environment.

If you are not comfortable with this Docker Playground you can get 5 Linux instances from any provider of your choice: DigitalOcean, VMware, Virtualbox, Google Cloud, Azure, AWS, etc.
All the steps will essentially be the same no matter the provider you are using.

If you use a different provider than Docker then you might need to install Docker engine if it is not already there.
In order to install docker engine please follow the instructions on this link:
- https://docs.docker.com/get-docker/

Why have I chosen 5 instances to create our cluster?

First of all because this number is the maximum allowed on the Docker Playground platform. I would have probably chosen 6 instances otherwise.

Secondly we need a number of instances that will suit our purpose of simulating a real production environment.
A real production environment needs High Availability for both the control plane (manager nodes) and data plane (worker nodes).

In order to ensure High Availability for the control plane we need at least 3 instances. 
This is because there is a database on the control plane that will store the cluster configuration and we need to guard the Quorum of the cluster.
When you have 2 instances available it is true that you can lose one of the instances without losing availability but you would lose the quorum.

How does the Quorum work?
Let us suppose that you have 3 instances of your database. 
They are permanently synchronized so that any change in any of the instances will immediately be transmitted to the rest of the database cluster blocking any further commit until this synchronization is completed.
This will ensure that all instances have identical content at any moment.

It might happen that one of the instances has a different content because of a network disruption or any other circumstancial reason.
In that case the leader instance of the cluster will enforce the content of the disturbing instance so as to match the majority of the cluster.
This will happen if we reboot any of the members of the plane (maybe for maintenance or any other reason).

It is important not to lose the Quorum because the Leader is "democratically" elected meaning that it will need simple majority (more than 50% of the members of the plane) in order to win the election.
If there is not enough Quorum and a new Leader cannot be elected the instances will remain splitted without possible automatic recovery (split brain).
This will happen if you lose 2 of 3 instances from the control plane.
The remaining instance will not be able to elect itself as a leader (because of the lack of quorum) and when the other 2 instances recover they will probably have different content and will not be able to agree in the election of the new leader.
That will cause a major disruption of the control plane and will need manual recovery.
The recovery will basically consist of forcing one the instances as the standalone leader and then joining the rest of the instances following this self-elected leader.
Normally you will choose as a leader the instance with the most recent changes on the database in order to lose as few data as possible.

The control plane use the Raft Consensus algorithm to control this process:
- https://en.wikipedia.org/wiki/Raft_(algorithm)

Three instances is the recommended amount of manager nodes for the control plane. 
A higher number will provide more protection (so you can lose more instances without losing quorum) but it will also increase the delay since any change in the database needs to be synchronize with the other members of the plane.
Docker highly recommends a maximum of 7 manager nodes for the control plane because the performance degradation would otherwise be unbearable.

Regarding the data plane (worker nodes) there is not specifically need for quorum unless you are also hosting a database cluster on your container platform.
In case you are hosting a database cluster (such as MySQL or Postgres for your custom applications) then you might also want to have at least 3 worker nodes for your data plane.

In our environment we have created 5 instances:
- 3 manager nodes (control plane)
- 2 worker nodes (data plane)

Let us initialize the Swarm cluster.
For this purpose click on the machine called node1 and run the following command: `docker swarm init`.
It will fail with the following message:
```
Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces
```

This means that the Docker engine has found multiple network interfaces and cannot choose one of them.
We need to help the engine choosing the appropriate address with the following command (instead of `192.168.0.18` you need to put the actual value of the IP address for your instance):
```
docker swarm init --advertise-addr 192.168.0.18
```

Now you will receive a successful message meaning that your instance is now the leading manager of the new cluster:
```
Swarm initialized: current node (ebxavqzk2yxayzzg03x0g97an) is now a manager.
```

The message continues with a clear instruction:
```
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4ge7nq62fbutfpsfevik6g6mrvbykcqj8ymdbmmj5lwuzli5no-7c14ew5u8m4jss5q71ifnk3c9 192.168.0.18:2377
```

We will need to run this command on the worker nodes.
First copy the command on the clipboard.
If you are using Docker Playground it is a bit tricky to get this copied: instead of the classical CTRL+C you need to type CTRL+INS after selecting the line.
In order to paste the content you will need to do SHIFT+INS instead of the classical CTRL+V.

Now select the next instance that is called node2 and paste the command.
Do the same with next instance called node3.
In both cases you will receive a successful message:
```
This node joined a swarm as a worker.
```

What is the meaning of the command that we used to join the worker nodes?
First of all `docker swarm join` will give instructions to the Docker engine to join the remote Swarm cluster.
There is a secret token in order to ensure that there are not strange nodes trying to join the cluster without our consent.
Finally there is the IP address of the leading manager node with the port where the API is listening.

We can now check the correct creation of the cluster running the following command on the manager node: `docker node ls`. 
If we try to run this command on any of the worker nodes it will fail with this message:
```
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state.
```

Now we have high availability for the data plane (2 worker nodes) but we still have only 1 single manager.
We can join more managers to the cluster with the following command on the manager node (node1): `docker swarm join-token manager`.
This will be the output of the command:
```
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4ge7nq62fbutfpsfevik6g6mrvbykcqj8ymdbmmj5lwuzli5no-9w44s6uatxkl7c9rl17hwaqkj 192.168.0.18:2377
```

It is again very clear. It is a very similar command as before with the same IP address and port but with a different token. 
Manager nodes have different tokens than worker nodes for security reasons.
We can run this command on the manager nodes (for example instances called node4 and node5).
In both cases we will receive this successful message:
```
This node joined a swarm as a manager.
```

We can check the correct deployment of the cluster running the following comand on any of the current manager nodes (control plane):
`docker node ls`.
You will see an output similar to the following:
```
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ebxavqzk2yxayzzg03x0g97an     node1      Ready     Active         Leader           20.10.0
3wxc5i2wdt5oxareitkhzxktz     node2      Ready     Active                          20.10.0
s6odbfb9ds5s686wjxivc63kz     node3      Ready     Active                          20.10.0
n8c36v0uhttbltgt3cl9yx8uj     node4      Ready     Active         Reachable        20.10.0
o46b74d2299tvc4ww37phiudb *   node5      Ready     Active         Reachable        20.10.0
```

In this list we can see one Leader (manager), two Reachable (also managers) and another two nodes without any manager status (workers).
All of them have Docker version 20.10.0 installed, "Ready" status and "Active" availability.

This is a perfectly valid highly available production container platform ready to deploy our containerized applications.

Let us do that in the next chapter.
