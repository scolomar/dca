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

As we have managers and workers in our cluster then we have two different types of high availability: for the management and for the workload.

The ensure the high availability of the workload we need at least two workers in the cluster.
When one worker is down for maintenance or by accident then the other worker will keep the workload available ensuring this way that our application is never down or unavailable.
Docker Swarm will balance the workload across the available nodes so that a failure in one node does not jeopardize the availability of our containerized applications.

If we have just one manager in the cluster then it will ensure its consistency but not the high availability.
Any failure or maintenance in this manager will require manual failover.
To ensure high availability and automatic resilience of the management we need to have more than one manager in our cluster.
The number of managers in the cluster is critical for its stability.
An even number of managers would easily result in a split-brain of the managers leading to an inconsistent global state of the cluster.

The consistency of the state of the cluster is guaranteed by the Raft Consensus Algorithm:
* https://en.wikipedia.org/wiki/Raft_(computer_science)
* http://thesecretlivesofdata.com/raft/

In order to ensure the correct election of the leader we will need to maintain the quorum or majority of the managers in the cluster:
* https://en.wikipedia.org/wiki/Quorum_(distributed_computing)

We will therefore need an odd number of managers to ensure the consistency of the state of the cluster.
A cluster with three managers will be resilient to the failure of only one manager but a failure of two managers will cause the cluster to become inconsistent and unrecoverable.
In that case only a manual recovery is possible.

To avoid this possibility Docker recommends a minimum of five managers so that the consistency of the cluster is guaranteed even in case of the failure of up to two managers since the surviving three managers will still be majority.

One could think then that a higher number of managers would even increase the stability of the cluster but there is another factor that invalidates that assumption.
Each manager that is added to the cluster increases exponentially the complexity and traffic of the management network inducing prohibitive delays.
That is why Docker recommends no more than seven managers.

In order to ensure high availability of the management plane we will therefore need a minimum of three managers and a maximum of seven. Docker oficially recommends five managers as the best practice for your cluster.
