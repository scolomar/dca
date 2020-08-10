## Chapter 5: Orchestration

### Section 1: Docker Swarm

Docker Swarm was released approximately two years later than Kubernetes.
Kubernetes is a very well known orchestrator that can handle any workload of containers across many worker nodes.
Kubernetes is nevertheless very complex to configure and manage.

Docker Swarm is also an orchestrator and as such it can also handle the lifecycle of any workload of containers across many worker nodes.
Docker Swarm is nevertheless much easier to configure and manage than Kubernetes.
It is also faster to react to any changes in the workload as well as more stable and lightweight.

We can think of Docker Swarm as a light, monolithic version of Kubernetes.

If you deploy Docker Swarm in your infrastructure instead of Kubernetes then you might save some money because you might need less hardware resources such as memory or CPU to handle the same amount of workload.
