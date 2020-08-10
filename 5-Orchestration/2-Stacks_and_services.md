## Chapter 5: Orchestration

### Section 1: Stacks and services

Once you have your cluster up and running you can deploy your business application.
A stack is a pile of microservices that will compose your containerized application.
You normally use a different stack for each independent application.

You need a compose file to deploy your stack.
This a manifest similar to a Kubernetes manifest.
The main difference between Docker and Kubernetes manifests is that in Kubernetes every object is described in its own manifest meanwhile Docker groups in the same compose file all the elements of the stack such as networks, volumes, services, secrets, configs, and any other fundamental part of your deployment.

When you are creating your Docker compose file to deploy your application it is good practice to check often the official reference: https://docs.docker.com/compose/compose-file.
It is the only source of trust when it comes to configure your deployment. 
The documentation is dynamic and is updated with any new feature or deprecation.
That is why the version of the compose file that appears in the manifest is so important to determine which version of the API it should reference.
The meaning of the various configuration elements will be different depending on the selected version so it is important to check whether a given feature is available in the version you are using.
For example `config` definitions are only supported in version 3.3 and higher of the compose file format as specified in the official documentation.

There is sometimes confussion regarding the name of the Docker compose file and the use of `docker-compose`. 
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


