## Chapter 7: Best practices

### Section 3: Backup and restore

It is important to keep a backup of your Docker container platform.
It is in general recommended to have a separated partition for /var/lib/docker folder.
This folder contains all the information about your Docker objects and their configurations including containers, images, volumes, networks and any other Docker object.

There are many good reasons for keeping this folder in a separated partition:
1. It might run out of disk space and therefore block your Docker deployment.
2. It will be easier to backup and restore if needed.
3. It might provide better security and even performance if the permissions and mounting options are properly configured.

The /var/lib/docker/swarm folder specificallly contains all the information regarding the Docker Swarm cluster as well as any object deployed or configured in the cluster.
If you keep a backup of this folder you can recover from a cluster disaster just restoring the folder on a new node and recreating a new cluster with the backup data.
