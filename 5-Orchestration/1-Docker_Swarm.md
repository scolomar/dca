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

