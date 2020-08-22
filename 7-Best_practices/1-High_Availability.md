## Chapter 7: Best practices

### Section 1: High availability

When you are running your containerized application in a real production environment you should be concerned about High Availability (HA).

HA is about having your application up and running on a 24/7 basis (24 hours / 7 days per week). At least this is what any business wishes.

The main purpose of running a business is making profit and your service application is supposedly the way to gain that profit. It is not good for the business to stop that service for maintenance or because of a failure.

How can then we achieve HA when running Docker?

The first thing you need is to duplicate your infrastructure.

If you only have one available machine and you need to reboot it to apply some security patches to the kernel then you will necessary stop your profitable application for an undetermined amount of time. This downtime window should be as little as possible but your business manager would prefer it to be zero.

If you have two machines in production then you can apply any necessary patches to one of the machines while the other is up and running and making profit to make your manager happy. After the first one is finished and perfectly rebooted you can then apply the patches to the other meanwhile the first one is already active. This way you will have your business always up and running.

Instead of planned or unplanned maintenance the downtime could also be caused by an unpredicted failure or any other possible reason. No matter what the cause is your application will always be available.

Two machines is the minimum number of machines to achieve HA for a stateless application. But this condition might change when the service you want to keep permanently up and running is not stateless but stateful. In the case of a database you also have to maintain the quorum. 

When there is a failure in any member of a database cluster it is possible that the content of the database will diverge from the rest of the cluster. When the failed member recovers and joins the cluster there will be a check of consistency of the database and it will show that there is a difference. In that case raft consensus (or a similar algorithm) applies. This way the majority of the cluster decides what is the right value for that discrepance in the database. And the majority imposes their decision to the divergent instances.

But majority means greater than 50% so that two machines is not enough to ensure HA for a database cluster. In that case the minimum value to keep the quorum is 3 machines. If one machines fails then you still have the other two machines to keep the quorum since 2 of 3 is more than 66%. But if two machines fail then your database will cause split-brain because of the lack of quorum. If that is a concern for you business then you should create a cluster of 5 machines in order to ensure HA with a fault tolerance of up to 2 machines. In any case you will need an odd number of machines.

Nevertheless take into account that a big number of instances in a database cluster will exponentially increase the delays since any change in any of the members needs to be immediately propagated to the rest of the cluster. This way a number greater than 7 will make the cluster unusable.

The recommended value will depend on the use case but definitely needs to be 3 or 5.

Now that we know that we need a minimum of 3 machines comes the question of where to locate them. If we place the 3 machines in the same network or data center and there is a general failure then we will have the same problem of lack of availability. So that it is recommended that the 3 instances are located in different networks and possibly different data centers to ensure the fault tolerance of the setup.

But there is still another concern. If the 3 instances are too separated from each other then unbearable delays will also occur that could eventually cause another split-brain in your cluster. So that keep them away from each other but not too much :-)

A nice solution if you are deploying your cluster in a cloud provider is to place each member of the database in a different availability zone but in the same region. This way your instances are separated enough from each other but still with a good connectivity to ensure smooth propagation of the changes across the cluster.

An example of such a construction is shown in the following picture:
https://github.com/secobau/proxy2aws/raw/master/proxy2aws.png

In the picture above there is a cluster spread across three different subnets. Each subnet can subsequently be placed in a different availability zone of the same region. 

In the picture you can also see that we have not only 3 worker machines but also 3 manager machines.

In a HA infrastructure we have to distinguish between worker and manager machines. The worker machines will be the ones which will be running your containerized application. By the contrary the manager machines will solely be responsible for the management and orchestration of the cluster. That includes keeping the cluster management database. 

What is the purpose of separating these two functionalities in two different classes of machines? 

When you are running your containerized application it might happen that a bug in the code or maybe a malicious attack could harm the host machine where the container is running. In case that this caused a failure of the machine you would not like it to affect the database that contains all the necessary data to manage the cluster. Such a failure could affect the quorum of your management cluster and ultimately ruin your business for an undetermined period of time.

In order to prevent that issue you want to run your containerized applications in the so-called worker machines meanwhile the manager machines will be exclusively dedicated to manage the whole system. If the management database of the cluster gets corrupted you will have a difficult problem to solve.

That is why we have our cluster set up with two different kinds of machines in the above example: 3 managers and 3 workers. This way we ensure HA of both the business application and the management of the cluster.

This is great. Now we have HA. But how can Docker handle this situation? Is the Docker engine capable of spreading the workload across a cluster of remote machines in differente availability zones?

The answer is positive if we enable the swarm mode. When swarm is initialized then the Docker engine is capable of managing our deployment across the cluster no matter how many machines we have.

Docker is then working as an orchestrator controlling the workload across multiple nodes. Docker is not the only available orchestrator. Kubernetes is another well known orchestrator but Docker Swarm is much easier to configure and deploy. Docker Swarm is also more stable, ligthtweight and faster to react to any change in the workload.
