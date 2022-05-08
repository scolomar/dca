## Chapter 5: Orchestration

### Section 2: Stacks and services

Once you have your cluster up and running you can deploy your business application.
A stack is a pile of microservices that will compose your containerized application.
You normally use a different stack for each independent application.

You need a Docker compose file to deploy your stack.
This a manifest very similar to a Kubernetes manifest.
The main difference between Docker and Kubernetes manifests is that in Kubernetes every object is described in its own manifest meanwhile Docker groups in the same compose file all the elements of the stack such as networks, volumes, services, secrets, configs, and any other fundamental part of your deployment.

When you are creating your Docker compose file to deploy your application it is good practice to check often the official reference:
- https://docs.docker.com/compose/compose-file.

It is the only source of trust when it comes to configure your deployment. 
The documentation is dynamic and is updated with any new feature or deprecation.
That is why the version of the compose file that appears in the manifest is so important to determine which version of the API it should reference.
The meaning of the various configuration elements will be different depending on the selected version so it is important to check whether a given feature is available in the version you are using.
For example `config` definitions are only supported in version 3.3 and higher of the compose file format as specified in the official documentation.

* There is sometimes confusion regarding the name of the Docker compose file and the use of `docker-compose`. 
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
2. Once you have set up the infrastructure then you will need to define the Dockerfile that will encapsulate your software inside a Docker image.
In this manifest you will execute all the necessary commands to ensure your application is correctly set up and configured as well as all the necessary dependencies are present in the resulting image.
3. The last step is to configure the Docker compose file where you configure the deployment of your containerized application.
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
2. To mount an external filesystem inside the container.

The reasons for doing so vary depending on the specific circumstances.
You could be for example interested in bypassing the copy-on-write system of the merged union filesystem of the container.
Or you could maybe interested in having property of a subfolder for security reasons (like when mounting /var/run in an external volume for example):
1. To have a persistent volume outside the lifecycle of the container to store for example a database.
2. To share data between containers.
3. Any other legitimate reason to use a volume as an external filesystem mounted inside the container.

Volumes are defined in the Docker compose file twice:
1. In the volume definition itself.
2. In the description of the container where it is specified the path where to mount the volume inside the container.

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

## Exercise

Let us deploy a few sample applications on our container platform.
We are going to use the Docker Swarm cluster created in the previous chapter.

In our first example we are going to deploy a very simple PHP application. Though the code that we are going to use is very simple, the principles would still be the same even when using more complex source code. In our case we are going to deploy PHP containers serving the following PHP code:
```
<?php phpinfo();?>
```

If we wanted to run this container using the standalone Docker engine then we would run the following code:
```
echo '<?php phpinfo();?>' | tee /tmp/index.php
docker run --detach --entrypoint php --name phpinfo --publish 8080 --restart always --user nobody --volume /tmp/index.php:/app/index.php:ro --workdir /app/ php -f index.php -S 0.0.0.0:8080
```
With the first command we are creating the PHP index file that we are going to serve.
The second line creates the container that will run the PHP embedded web server.

Let us analyze the different parameters that compose the command line:
1. The command `docker run` will call the Docker engine to create the container.
2. The option `detach` will run the process in the background. Otherwise the terminal would be blocked by the output of the main process that is running in the container.
3. The option `entrypoint` defines the binary the will be run in the container as the main process (`/usr/local/bin/php` in our case). Normally there will be a `PATH` environment variable defined in the container so that we will not need to specify the absolute path of the binary: `php` will be enough to describe the location of the entrypoint.
4. The option `name` will give a name to our container. This will be useful for troubleshooting and when referring to the container at any moment in the future.
5. The option `publish` will specify a port in the container network that we want to map into the host network creating what is called (in Kubernetes) a "nodePort". This host port will be randomly chosen from a specific range (starting at `32768` to avoid conflicting with Kubernetes own range). It is possible to hardcode the value of the "nodePort" but it is highly recommended not to do so in order to avoid conflicting situations. Thanks to this mapping we will be able to connect to the container port targeting the random host port on the host network. In this example we will be able to access the container web server pointing to the host network (since the container network is unaccessible by default).
6. The option `restart` will keep our container always up and running automatically recovering after any kind of failure or shutdown.
7. The option `user` will specify the user that we want to take ownership of the process running at the entrypoint. In our example we want the user `nobody` to take ownership of the `php` process. This is obviously done in order to improve the security of the container. It is also recommended not to use any system username because those users could be used for other functions of the Linux system. Following this recommendation `nobody` would not be a good choice and it would be necessary to create a dedicated user to run the entrypoint binary.
8. The option `volume` will mount the file that we have just created on our local filesystem (`/tmp/index.php`) into the container on the specified location (`/app/index.php`). The cointainer location does not need to exist (in this case the folder `/app/` will be automatically created when starting the container).
9. The option `workdir` will specify the working directory where we want to run the process. It is typically chosen the location of the data.
10. After the last option comes the name of the Docker image (`php` in our case). It is an abreviation though the full name would be: `index.docker.io/library/php:latest` which gets simplified with the short name `php`. We could have used for example `php:alpine` instead (or in its full name: `index.docker.io/library/php:alpine`.
11. After the name of the Docker image comes the list of arguments for the entrypoint. If we wanted to run the application without containerization this would be the command: `php -f index.php -S 0.0.0.0:8080`. Therefore the arguments of the entrypoint are: `-f index.php -S 0.0.0.0:8080`. It is following PHP syntax and basically means the file to publish (`-f index.php`) and the port and IP where to listen (`-S 0.0.0.0:8080`). (You can get more information running `php -h` or checking the PHP documentation).

We have just described how to deploy the application with a standalone Docker engine.
This can be very useful for testing or even deploying an application in a scenario where we do not need high availability.
We need to understand that the application will be stopped when the Docker enginer is down because of failure or maitenance purposes.
In order to ensure high availability of our application we need to create a Docker service that will be migrated to another Docker engine (that is to another host) if our current host is down for any reason.
Let us see how we can deploy our application on high availability mode.
We can do that with the command line or we can use a Docker compose file.
These would be the commands to deploy our PHP service with high availability:
```
echo '<?php phpinfo();?>' | docker config create index.php -
docker service create --config source=index.php,target=/app/index.php,mode=0400,uid=65534 --entrypoint php --mode replicated --name phpinfo --publish 8080 --read-only --replicas 2 --restart-condition any --user nobody --workdir /app/ php -f index.php -S 0.0.0.0:8080
```

In the first command we have created a Docker config. It is a Docker object that will store our configuration file as a key-value record in Docker Swarm database.
In case we needed secrecy we could have created a Docker secret instead. This would have been the command to create the Docker secret:
```
echo '<?php phpinfo();?>' | docker config create index.php -
```

In the second command we have created the Docker service with the following optins:
1. The option `config` specifies the Docker config we want to mount inside the container including the permissions and user ID (`65534` for the user `nobody`) to access the mounted file.
2. The option `entrypoint`specifies the entrypoint binary for the application we want to run inside our container.
3. The option `mode` indicates whether we want to deploy the service as replicated or global. Replicated is the default mode. Global means to deploy exactly one replica per node.
4. The option `name` gives a name to identify the service.
5. The option `publish` will map a random port on the host (in the range starting at `30000`) with the specified container port  of the replicas (`8080` in our case).
6. The option `read-only` will mount the container filesystem of the replicas in read-only mode which can be very useful for security purposes.
7. The option `replicas` indicates the number of containers we want to deploy for our service. They will be evenly distributed across the nodes in a best-effort way with no guarantee of symmetry.
8. The option `restart-condition` specifies the restarting condition for the containers in case of failure or any other reason.
9. The option `user` indicates the user that will take ownership of the main process.
10. The option `workdir` specifies the working directory where we want to run the main process.
11. After all these options come the name of the Docker image (`php`) which is a simplification of `index.docker.io/libary/php:latest`.
12. After the name of the image comes the arguments for the entrypoint: `-f index.php -S 0.0.0.0:8080`.
13. You can use the command `docker service create --help` to explore other possible options.

Now that we have created our Docker service to deploy our containerized application there are some useful commands to troubleshoot any issues that might appear:
1. `docker service ls` will list the currently deployed services.
2. `docker service logs` will fetch the logs of a service or task. The command will actually provide the output of the main application (with PID number 1) running on the deployed containers. If the running application does not provide any output then the logs will be empty.
3. `docker service inspect` will display detailed information on one or more services which can be useful for troubleshooting failing services.
4. `docker service ps` will list the tasks of one or more services. If there is any failure in any of the deployed containers we will visualize it with this command.
5. `docker service rm` will let us remove one or more services.
6. `docker service scale` will scale one or more replicated services (it will obviously not work for global services).
7. Please check other options with the command `docker service --help`. It is important to learn to use the embedded help of Docker commands.

Let us deploy a misconfigured service in order to provoke a failure and be able to debug it.
The following command will fail to deploy the service because there is an error in one of the parameters:
```
docker service create --config source=index.php,target=/app/index.php,mode=0400,uid=nobody --detach --entrypoint php --mode replicated --name phpinfo-failure --publish 8080 --read-only --replicas 2 --restart-condition any --user nobody --workdir /app/ php -f index.php -S 0.0.0.0:8080
```

With the command `docker service ls` we will see that there is a service with `0/2` replicas meaning that it has not been able to correctly run any of the replicas.
We can see more details with the command `docker service ps phpinfo-failure --no-trunc` that will show the reason for the failing container: "starting container failed: strconv.Atoi: parsing "nobody": invalid syntax".
The service was misconfigured because instead of `uid=65534` we wrote `uid=nobody`.

Now it is time to learn how we can deploy this same PHP service using a template instead of a command line.
The advantages of using YAML templates instead of command lines are many:
1. We will be able to track any changes through a Version Control System (like git).
2. It will be easier to configure a CI/CD pipeline based on that template.
3. It will be easier to share with other teams and reproduce any deployment.
4. It will be more reliable to track changes in the template than the history of the past run commands.

In order to create a service with a YAML template we will prepare a Docker compose file that will be sent to the Docker engine whose API will process the request and generate the necessary objects.
The result will be equivalent to having used the command line for that purpose.

This is an example of a valid Docker compose file to deploy the sample application:
```
# docker-compose.yaml
# THIS IS THE LIST OF DOCKER CONFIGS WE WANT TO APPLY
configs:
  # THIS IS THE DOCKER CONFIG WE CREATED IN A PREVIOUS STEP
  index.php:
    # THIS WILL INVOKE THE PREVIOUSLY CREATED DOCKER CONFIG
    external: true
# THIS IS THE LIST OF SERVICES WE WANT TO CREATE
services:
  # THIS IS THE NAME OF THE SERVICE
  phpinfo:
    # THESE ARE THE ARGUMENTS OF THE ENTRYPOINT
    command:
      - -f
      - index.php
      - -S
      - 0.0.0.0:8080
    # EQUIVALENT TO: --config source=index.php,target=/app/index.php,mode=0400,uid=65534
    configs:
      - 
        mode: 0400
        source: index.php
        target: /app/index.php
        uid: '65534'
    # EQUIVALENT TO: --mode replicated --replicas 2 --restart-condition any
    deploy:
      mode: replicated
      replicas: 2
      restart_policy:
        condition: any
    # EQUIVALENT TO: --entrypoint php
    entrypoint: php
    # THIS IS THE NAME OF THE DOCKER IMAGE
    image: php
    # EQUIVALENT TO: --publish 8080
    ports:
      - 8080
    # EQUIVALENT TO: --read-only
    read_only: true
    # EQUIVALENT TO: --user nobody
    user: nobody
    # EQUIVALENT TO: --workdir /app/
    working_dir: /app/
# THIS IS THE DOCKER API VERSION THAT WE WANT TO USE FOR THIS DEPLOYMENT
version: "3.8"
```

Once we have saved the previous file with the name `docker-compose.yaml` (for example) we can then deploy the stack (composite application) with the following command:
```
docker stack deploy --compose-file docker-compose.yaml PHPINFO
```

Then we can use the following commands to investigate the deployment:
1. `docker stack ls` to list the deployments.
2. `docker stack services PHPINFO` to list the services that have been deployed.
3. `docker stack ps PHPINFO` to retrieve more details about the containers deployed by our stack.
4. `docker stack deploy --compose-file docker-compose.yaml PHPINFO` to redeploy the stack after any changes on the template.
5. `docker stack rm PHPINFO` to remove the deployment.

Of course we can still use any other Docker command to troubleshoot any issues with the deployed stack (like for example):
1. `docker ps` to see the list of containers.
2. `docker logs` to check the container logs.
3. `docker inspect` to view more details about a Docker object.
4. `docker service ls` to list the deployed services.
5. `docker service logs` to see the logs of a service.
