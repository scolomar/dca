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
