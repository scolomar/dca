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
This is a specific issue of Kubernetes that needs to be taken into account so as not to confuse the persistence of the volume outside the container and the persistence of the PersistentVolume outside the Pod.

## Exercise

In the following exercise we are going to practice with persistent storage for our containers.
Let us start with a very simple example where we will create a basic container and write a new file into it.
```
docker run --detach --name test --tty busybox
docker exec test touch hello-world.txt
```
With the previous commands we have create an empty file named `hello-world.txt` inside de the container filesystem.
We can see the changes in the filesystem with the following command:
```
docker diff test
```
This is the output of the previous command:
```
A /hello-world.txt
```
The `A` at the beginning of the line means that the file has been "added" (A) in that location.
That is the location inside the container filesystem but what is the actual location of this new file on the host filesystem?
We can find that information with the following command:
```
find /var/lib/docker/overlay2/ -type f | grep hello-world.txt
```
This is the output of the command:
```
/var/lib/docker/overlay2/48f84f672c67eccd39373dc9003e6a8c27139024bd0e990cf682572222c375e3/diff/hello-world.txt
/var/lib/docker/overlay2/48f84f672c67eccd39373dc9003e6a8c27139024bd0e990cf682572222c375e3/merged/hello-world.txt
```
We can see in the output two subfolders: `diff` and `merged`.
- `diff` folder will show you the differences in the container filesystem compared with the original Docker image.
- `merged` contains the "merged" filesystem: that is the merge of the original Docker image and the different changes of the container filesystem.

Let us see what happens with these changes when we stop the running container:
```
docker stop test
docker diff test
find /var/lib/docker/overlay2/ -type f | grep hello-world.txt
```
We can still view the changes to the filesystem with `docker diff` and find `hello-world.txt` in `/var/lib/docker/overlay2/` subfolder:
```
$ docker stop test
test

$ docker diff test
A /hello-world.txt

$ find /var/lib/docker/overlay2/ -type f | grep hello-world.txt
/var/lib/docker/overlay2/48f84f672c67eccd39373dc9003e6a8c27139024bd0e990cf682572222c375e3/diff/hello-world.txt
```

But what happens when we remove the container?
Then everything disappears: the container will be erased from the system and it will not be possible to recover `hello-world.txt` as we can see after running the following commands:
```
docker rm test
docker diff test
find /var/lib/docker/overlay2/ -type f | grep hello-world.txt
```
These are the outputs of the previous commands:
```
$ docker rm test
test

$ docker diff test
Error response from daemon: No such container: test

$ find /var/lib/docker/overlay2/ -type f | grep hello-world.txt
```

Therefore the data will be safe in the container as far as we do not remove the container.
This might work in some use cases but even though the data is not lost the performance of the container is diminished because of the Copy-on-Write strategy.
The host operating system will merge any changes in the container filesystem before the containerized application can access the modified files.

There is another way to persist the data without polluting the container filesystem: Docker volumes.
When using Docker volumes our data will be stored in a subfolder of `/var/lib/docker/volumes/` outside the container filesystem and fully bypassing the Copy-on-Write strategy:
```
docker run --detach --name test --read-only --tty --volume myvolume:/mydata/ busybox
```
In this case `myvolume` will be the name of the Docker volume that will be created to store the data. 
The location inside the container: `/mydata/` will be created on-the-fly if it does not previously exist.
I can even use the option `read-only` to increase the security of my container: given that the data will be written somewhere outside the container filesystem I can run it in read-only mode.

Let us see the content of my new volume:
```
$ docker exec test ls /mydata/ -l
total 0

$ find /var/lib/docker/volumes/myvolume/
/var/lib/docker/volumes/myvolume/
/var/lib/docker/volumes/myvolume/_data
```
There is a subfolder on the host filesystem called `_data` which is originally empty and will contain all my data.

Let us create a new file:
```
$ docker exec test touch hello-world.txt
touch: hello-world.txt: Read-only file system
```
The error message is warning us that the container file system is in read-only mode.
In order to create the new file we need to point to the external volume mounted inside the container:
```
docker exec test touch /mydata/hello-world.txt
```
Now I will be able to see the changes:
```
$ docker exec test ls /mydata/ -l
total 0
-rw-r--r--    1 root     root             0 May 13 02:29 hello-world.txt

$ find /var/lib/docker/volumes/myvolume/
/var/lib/docker/volumes/myvolume/
/var/lib/docker/volumes/myvolume/_data
/var/lib/docker/volumes/myvolume/_data/hello-world.txt
```
Nevertheless `docker diff` will not be much useful in this case since this command only shows the changes made to the container file system and the data is now located on another file system (external to the container):
```
$ docker diff test
A /mydata
```
It shows that a new folder `/mydata` has been created but it does not show the content of the data since it is actually located outside the container.
