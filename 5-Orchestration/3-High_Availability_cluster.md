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
