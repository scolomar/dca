## Chapter 4: Storage and Volumes

### Section 2: Best practices

What are the recommended best practices regarding storage? 

There are many different customers with different use cases. 
Each specific situation will require a specific approach. 
There are nevertheless some common principles that may apply in general.

The main principle is that of immutability. The Docker image is (and should be) immutable.
This is one of the main advantages and revolutionary concepts of Docker: 
"If it works in my laptop then it will work in a production environment because the Docker image is exactly the same".

But there is some content that cannot (and should not) be included in the Docker image. 
For example the configuration of the environment: the proxy configuration or the password for the database should not have the same value in development and production.
You need a way to inject that content in the filesystem of the container.
The Docker image is immutable so that you cannot modify the content.
It would be a terrible idea to include that configuration inside the Docker image because then it would be forever bound to that environment.

The container is writeable but it is not considered a good practice to modify its content. 
The merged filesystem is created by the superposition of all the layers that compose the Docker image and the container.
This operation costs a high price in terms of CPU and memory for the operating system of the worker machines.
It is well known that a filesystem consisting of too many layers will reduce the performance of the container.
What is then the best solution?

One recommended solution is to inject the configuration data in an external volume.
This external volume will be mounted inside the container but that will not affect its performance because it will be a different filesystem.

## Exercise

When you create a container it is better to mount the filesystem in read-only mode: this will improve not only the security but also the performance of the container.
The security will increase because the filesystem will not be writable in case the container is compromised.
The performance will increase since we will avoid the penalty of the merge caused by the Copy-on-Write strategy.
We can create a container in read-only mode with this command:
```
docker run --detach --name test-readonly --read-only --tty busybox
```

But there are circumstances where our containerized application needs to write stuff to the filesystem.
For that purpose we can attach a Docker volume to the read-only container:
```
docker run --detach --name test-volume --read-only --tty --volume myvolume:/mydata/:rw busybox
```
The option `:rw` after the volume mount path means that the filesystem will be writeable.
It is recommended to mount the volume in read-only mode unless absolutely necessary adding the option `:ro` after the mount path:
```
docker run --detach --name test-volume-readonly --read-only --tty --volume myvolume:/mydata/:ro busybox
```
We can attach as many volumes as needed:
```
docker run --detach --name test-volumes --read-only --tty --volume vol1:/data1/:rw --volume vol2:/data2/:rw --volume vol3:/data3/:ro busybox
```

It is typically considered a good practice to include any configuration or data in a separated volume outside the container filesystem both for security and performance reasons.
