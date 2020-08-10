## Chapter 5: Orchestration

### Section 1: Stacks and services

Once you have your cluster up and running you can deploy your business application.
A stack is a pile of microservices that will compose your containerized application.
You normally use a different stack for each independent application.

You need a Docker compose file to deploy your stack.
This a manifest very similar to a Kubernetes manifest.
The main difference between Docker and Kubernetes manifests is that in Kubernetes every object is described in its own manifest meanwhile Docker groups in the same compose file all the elements of the stack such as networks, volumes, services, secrets, configs, and any other fundamental part of your deployment.

When you are creating your Docker compose file to deploy your application it is good practice to check often the official reference: https://docs.docker.com/compose/compose-file.
It is the only source of trust when it comes to configure your deployment. 
The documentation is dynamic and is updated with any new feature or deprecation.
That is why the version of the compose file that appears in the manifest is so important to determine which version of the API it should reference.
The meaning of the various configuration elements will be different depending on the selected version so it is important to check whether a given feature is available in the version you are using.
For example `config` definitions are only supported in version 3.3 and higher of the compose file format as specified in the official documentation.

* There is sometimes confussion regarding the name of the Docker compose file and the use of `docker-compose`. 
Docker Swarm is an orchestrator integrated in the Docker enginer but `docker-compose` is a separate application written in Python.
Docker Swarm is an orchestrator in the sense that it can handle the deployment of your workload across a network of worker nodes. 
That is not the case of `docker-compose` that can only deploy in a standalone machine using only basic Docker commands. 
Many of the features available to Docker Swarm such as configs and secrets are not available to `docker-compose`.
So that `docker-compose` is a nice tool that can be useful for a developer that wants to test a containerized application in his laptop or in a standalone machine but will be totally useless in a production environment where we want high availability by default.
My recommendation for developers is always to use Docker Swarm even when you are deploying in your own laptop and therefore deprecate the use of `docker-compose`.
Any developer can easily set up a Docker Swarm in his laptop with the command `docker swarm init`.
You do not need to add any other worker or manager to the cluster that will be in this case a standalone cluster consisting of one single machine.
This approach will allow the developer to use in his laptop or testing environment the same configuration options such as Docker configs and secrets that will be used in the production environment.

When you want to deploy a containerized application you need three main manifests:
1. The first one is the manifest to create your infrastructure.
You can use CloudFormation templates for AWS, Azure Resource Manager templates, Google Cloud templates, Vagrant, Terraform, Openstack or any other kind of template to deploy your infrastructure in the provider of your choice. Be it on premises or in the cloud.
1. Once you have set up the infrastructure then you will need to define the Dockerfile that will encapsulate your software inside a Docker image.
In this manifest you will execute all the necessary commands to ensure your application is correctly set up and configured as well as all the necessary dependencies are present in the resulting image.
1. The last step is to configure the Docker compose file where you configure the deployment of your containerized application.
In this last manifest you describe all the necessary objects for the correct execution of your application like config files and secrets, volumes and networks.

In the Docker compose file you define how to deploy the Docker images that you have built following the Dockerfile manifest on top of the infrastructure you have built following the CloudFormation or Terraform template of your choice.
This definition includes basic elements like networks, volumes, secrets, configs and services.

The principle that should apply here is that any of these three manifests needs to be as generic as possible avoiding to hardcode specific parameters inside the manifest.
We will use for example `git clone` to download any necessary software or script in order not to include specific code inside the manifest.
We will also try to use variables, arguments and parameters whenever possible to avoid hardcoding the manifests with specific values.

One of the main problems to solve when deploying your containerized applications is how to configure the specific requirements for the three main differents stages or environments: development, testing and production.
Different environments will obviously need different configuration parameters. The proxy definition for production will probably be different to that of development. The database credentials should definitely be different in any of these environments.
To solve this problem we will use env files, Docker configs and secrets. These elements are equivalent to their counterparts in Kubernetes configmaps and secrets.

We will use env files and Docker configs to provision non confidential specific configuration to our Docker images.
The env files consist of simple text files with a set of environment variables declarations that are provisioned to the services upon creation and provisioned as environment variables in each container deployed for that specific service.
This means that executing a terminal inside any of those containers would allow us to view the value of those environmental variables with the command `printenv`.

Docker configs are also defined as simple text files containing any desired configuration or even scripts but they are managed differently by Docker Swarm.
They are stored as config objects by the Raft database of the managers and provisioned as files mounted at a specific path inside the container.
This mountpoint will use `tmpfs` so that the file will be mounted in the memory of the container (feature only available in Linux machines) instead of using the disk of the worker machine.
The config objects will be nevertheless stored in plain text in the Raft database of the managers.

If you need to provision confidential information to your containers then it is preferable to use Docker secrets. 
They are equivalent to Docker configs but in this case they are stored encrypted in the Raft database of the managers.
They can provision any kind of text files, scripts or even binary files to the target containers.
The only limit is the size with a maximum of 500 kB.

The correct use of Docker configs and secrets would be the following: any specific configuration for the environment such as the proxy or database credentials will be described in a separate file through a Docker config or secret definition. This way the deployment of the same Docker container in testing or production will not vary anyone of the manifests. The Dockerfile will be the same in testing and production as well as the Docker compose file. The only difference will be the content of the Docker config and secret that will contain the actual credentials for the testing or production environment respectively.
This method will ease the implementation of Continous Integration and Continuous Delivery (CI/CD) pipelines.

Given the limited size for the Docker configs and secrets we will use volumes when we want to provision any file of bigger size.

Volumes will be used for any of these use cases:
1. To provision any configuration to the containers that does not fit in a Docker config or secret.
1. To mount an external filesystem inside the container.
The reasons for doing so vary depending on the specific circumstances.
You could be for example interested in bypassing the copy-on-write system of the merged union filesystem of the container.
Or you could maybe interested in having property of a subfolder for security reasons (like when mounting /var/run in an external volume for example).
1. To have a persistent volume outside the lifecycle of the container to store for example a database.
1. To share data between containers.
1. Any other legitimate reason to use a volume as an external filesystem mounted inside the container.

Volumes are defined in the Docker compose file twice:
1. In the volume definition itself.
1. In the description of the container where it is specified the path where to mount the volume inside the container.

There is little difference between the Docker compose file and the Kubernetes manifest for the Deployment object. Both of them have similar specifications.

Besides the volumes we will also describe the networks in the Docker compose file.
It is important to take into consideration that two services (sometimes called microservices) will not be able to talk to each other unless they are in the same Docker network.
One service can nevertheless be attached to as many networks as needed.
So that for example if there is a microservice running Tomcat that wants to talk to another microservice runing MySQL then both of them need to be attached to the same network otherwise Docker will by default block any traffic between them.
Docker networking is secure by default.
This is the opposite of Kubernetes networking.
In Kubernetes any microservice can by default talk to any other microservice in the same namespace.
Only setting up a Network Policy can block the traffic between Pods.

Services are also described in the Docker compose file.
They are sometimes called microservices by some users because they represent the decomposition of a monolythic application into different "microservices" but the official definition is "services" both in Docker and Kubernetes.
Services are actually internal load balancers that will balance the requests to the service through a set of targets that are the actual containers running the application (Pods in the case of Kubernetes) and listening on the specified port.
The name of the service will represent the full qualified domain name (FQDN) of the load balancer.
The IP of the load balancer (also called VIP: Virtual IP) will be fixed during the whole life of the service (the same thing is called Cluster IP in Kubernetes) and it will be accessible only from the Docker network attached to the service unless we also publish the port in the network of the host machine with the option `--publish`.
Docker Swarm will set up a (configurable) healthcheck to determine whether the target container is alive otherwise remove it from the list of available targets for the load balancer of the service.

In the definition of the service we also specify the conditions for the deployment such as the number of replicas, the placement conditions, the resource limits to be applied, the Docker image to be run, as well as any other specification. It is interesting to be noticed that in Kubernetes we separate this definition in different objects: one for the service, another for the deployment, another one for the external routing, another one for the persistent volume, and so on.
As Docker Swarm is a monolythic solution everything is integrated in the same definition of the Docker compose file though the characteristics of each component are very similar to their counterparts in Kubernetes.

Once we have defined the conditions for the deployment in a section of the service definition then Docker Swarm will schedule the required replicas.
The deployment of an individual replica (that is a container) will need to be scheduled in a specific node with enough resources.
The Docker task is the scheduling of one replica in an available node.
Once a task has been assigned to a node that will never change.
If the deployment of a replica fails then that task will be shut down and replaced by another task but tasks are never restarted.
The same issue happens in Kubernetes with pods (which represent the scheduling of one replica to an available node).
If a pod fails for some reason then it is replaced by a new one but never rescheduled or restarted.
There is therefore a clear similarity between Docker tasks and Kubernetes pods.
The main difference between a Docker task and a Kubernetes pod is that a Docker task can only spin up one individual container (replica) meanwhile a Kubernetes pod can host more than one container.
