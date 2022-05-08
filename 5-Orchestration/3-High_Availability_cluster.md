## Chapter 5: Orchestration

### Section 3: High Availability cluster

As we have managers and workers in our cluster then we have two different types of high availability: for the management and for the workload.

The ensure the high availability of the workload we need at least two workers in the cluster.
When one worker is down for maintenance or by accident then the other worker will keep the workload available ensuring this way that our application is never down or unavailable.
Docker Swarm will balance the workload across the available nodes so that a failure in one node does not jeopardize the availability of our containerized applications.

If we have just one manager in the cluster then it will ensure its consistency but not the high availability.
Any failure or maintenance in this manager will require manual failover.
To ensure high availability and automatic resilience of the management we need to have more than one manager in our cluster.
The number of managers in the cluster is critical for its stability.
An even number of managers would easily result in a split-brain of the managers leading to an inconsistent global state of the cluster.

The consistency of the state of the cluster is guaranteed by the Raft consensus algorithm:
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

## Exercise

In this exercise we are going to test the resilience and failover of a Docker Swarm cluster.
For this purpose we are going to use the Docker Playground platform as explained in previous chapters.
Before proceeding go the following link and access with your Docker account:
- https://labs.play-with-docker.com/

Then create a highly available cluster with 3 managers and 2 worker nodes as described in Section 5.1: Docker Swarm (Exercise).
It should take you less than 5 minutes.

Let us deploy a sample application after the cluster is finished:
```
echo '<?php phpinfo();?>' | docker config create index.php -
docker service create --config source=index.php,target=/app/index.php,mode=0400,uid=65534 --constraint node.role==worker --entrypoint php --mode replicated --name phpinfo --publish 8080 --read-only --replicas 1 --restart-condition any --user nobody --workdir /app/ php -f index.php -S 0.0.0.0:8080
```
I have on purpose deployed one single replica so as to show how Docker will manage to keep one instance of our application permanently available.
The `constraint` option will force Docker to deploy the container on a worker node (instead of a manager node).

First we are going to manually kill the container and see how Docker engine reacts to this situation.
In order to kill the container we first need to locate it with the following command: `docker service ps phpinfo`.
The output of this command will show us on which node is the container actually running.
In my case it says it is running on Node2 so I will jump to that node and identify the running container with the command `docker ps | grep phpinfo`.
I will kill the container running the following command: `docker rm --force $( docker ps | grep phpinfo | awk '{ print $1 }' )`.
