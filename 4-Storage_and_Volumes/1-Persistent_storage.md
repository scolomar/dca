## Chapter 4: Storage and Volumes

### Section 1: Persistent storage

How does storage work for a container?

The Docker image is the primary filesystem that will be available to the container.
This means that any binary, library or dependency that any process running inside a container might need should be contained in the Docker image.
But the Docker image is a read-only filesystem.
How can then a process running inside a container write anything to the filesystem?
For that purpose the container will mount a writable layer that will be superposed to this read-only image.
This layer is normally called the "container layer".

When a process running inside a container needs to modify, create or delete any file of the image filesystem then it will create a copy of the file inside the writable layer and then it will modify or delete that copy of the file instead of the original read-only version.
This is what is called the Copy-on-Write (CoW) strategy.

Let us suppose that we have a web server deployment with ten instances of the Apache web server running on a host machine.
Those ten instances will use the same read-only image therefore it is not necessary to replicate the Docker image.
One read-only image is enough to instanciate as many containers as needed.
What will instead be replicated is the container writable layer.
There will be as many container layers as container replicas we have deployed.
In our case there will be one read-only image of the Apache web server and ten writable container layers.
Any process inside a container that needs to write something to the filesystem will use the container layer since the Docker image is immutable.

The Union Filesystem is the underlying technoloy to make this happen. 
It will create a superposition of the writable and non writable layers that will be exposed as the actual filesystem of the container.
Any intention to modify this filesystem will use the Copy-on-Write strategy to access any file from the internal read-only layers.
After copying the file to the writable layer then it will be modified or deleted.
Only the new files will be directly created on the writable layer.

As we can imagine this method will reduce the performance of the container since any attempt to read or write any file within the filesystem will have to first determine in which layer is located the latest version of the file that wants to be accessed.

It is normally considered a good practice to avoid the use of this Union Filesystem as much as possible to improve the performance of our containers.
For that purpose it is recommended to have as few read-only image layers as possible so as to reduce the burden of this superposition calculation.

But sometimes it is absolutely necessary to write data to the filesystem like for example in the case of a database application such as MySQL or PostgreSQL.
In these cases we will use a totally different object to circumvent the Union Filesystem: the Docker volume.

A Docker volume is a filesystem totally independent of the Docker image and the container layer that will be mounted inside the container in a specified path.
There are many types of Docker volumes.
They can be located in any path of the host machine or even in memory using the tmpfs mount.
Using specific volume plugins will even allow you to use external storage from the cloud, virtual platforms or many other available options from third party providers.
In any case a Docker volume will allow you to avoid the Copy-on-Write system improving the I/O performance of the container.

The Docker volume will also persist the lifecycle of the container. 
After a container is stopped the main process is removed from the memory of the host machine but the writable container layer is untouched.
Therefore you can restart a stopped container because the full state of the container is kept in this writable layer.
When a container is removed then the container layer is also removed and the container is unrecoverable.
But the volume is untouched by default.
You need to force it if you want to remove any volume associated with a given container otherwise the removal of a container will not imply the removal of any attached volume.
Therefore the volume will persist the removal of the container.
This strategy is normally preferred when dealing with databases so that the official Docker image of any typical database like MySQL or PostgreSQL will by default create a volume to store the data.

Volumes can also be shared among different containers.
This a very useful feature but also very dangerous since a volume is a point of ingress to the container breaking therefore the isolation of the filesystem mount namespace.
If a container is compromised and has permissions to modify a volume that is shared across many containers then the attack will potentially compromise other containers.
It is considered a good practice to mount the volumes in read-only mode when possible in order to avoid this danger.

In case you are using Kubernetes as your orchestrator then you need to consider how volumes are managed inside the Pods.
Each Pod can use the volumes defined so far that will be shared by the containers running inside the Pod and will persist the data even after the removal of the container.
But if the Pod is removed then the volumes attached to the containers inside the Pod will also be removed.
That is why Kubernetes has another object called PersistentVolume that is external to the Pod and will persist after the removal of the Pod.
This is a specific issue of Kubernetes that needs to be taken into account so as not to confuse the persistence of the Volume outside the container and the persistence of the PersistentVolume outside the Pod.

