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
I will kill the container with the following command on Node2:
```
docker rm --force $( docker ps | grep phpinfo | awk '{ print $1 }' )
```

Now jump to any manager node to observe the reaction of Docker Swarm.
If we again run `docker service ps phpinfo` we can see that there are two tasks in the list: one is running and the other failed a few seconds ago with an error message: `non-zero exit` caused by the `docker rm` command.
By the difference in seconds between the failed container and the running container we note that it just took about 5 seconds to recover the container from the forced failure.
Awesome!

The recovered container will be recreated on the same node as the original container since when a task is assigned to a worker node that assignment is permanent until the node becomes unavailable.
Our next experiment is to fail the node and see the reaction of Docker Swarm.
In my case the task is running on Node2 so that I am going to shut that node down to provoke a reaction.
So I jump to Node2 and click the "DELETE" button.
Shortly after deleting the node Docker Swarm has detected tha failure as you can see with the command `docker node ls` whose output will show that the node is `Down`.
After approximately 1 minute of failure Docker Swarm has migrated the task to another worker node (in my case Node3).
It has taken more time to react since Docker Swarm is configured to be more conservative regarding node failures in order to avoid a harmful stampede.

We have so far proved the automatic failover of a killed container and a deleted worker node.
What about the control plane?
Will Docker Swarm resist the failure of one manager node?
As we have created a highly available cluster with 3 manager nodes nothing should happen after deleting one of the managers.
So jump to any manager node and click the "DELETE" button.
Docker immediately detects the node as "Unreachable" as described in the output of `docker node ls`.
And the running service remains completely unaffected as can be checked with the command `docker service ps phpinfo`.
Of course we can visit the running application clicking on the link with the port number at the top of the web page of the Docker Playground dashboard.

But if we lose a second manager node then the situation becomes ugly.
Let us jump a second manager node and click again on the "DELETE" button.
Docker Swarm will lose quorum at the control plane and it will become unaccessible.
We will receive the following error message when trying to connect to the manager to query for the running services or nodes:
```
The swarm does not have a leader. It's possible that too few managers are online. Make sure more than half of the managers are online.
```

This can be a difficult situation that we will always want to avoid in a production environment.
Nevertheless the container will continue running without any problem though disconnected from the control plane (so unmanaged and vulnerable to any kind of disruption).
