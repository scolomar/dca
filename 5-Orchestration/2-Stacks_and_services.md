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

When you want to deploy a containerized application you need three main manifests:
1. The first one is the manifest to create your infrastructure.
You can use CloudFormation templates for AWS, Azure Resource Manager template, Google Cloud templates, Vagrant, Terraform, Openstack or any other kind of template to deploy your infrastructure in the provider of your choice. Be it on premises or in the cloud.
1. Once you have set up the infrastructure then you will need to define the Dockerfile that will encapsulate your software inside a Docker image.
In this manifest you will execute all the necessary commands to ensure your application is correctly set up and configured as well as all the necessary dependencies are present in the resulting image.
1. The last step is to configure the Docker compose file where you configure the deployment of your containerized application.
In this last manifest you describe all the necessary objects for the correct execution of your application like config files and secrets, volumes and networks.

In the Docker compose file you define how to deploy the Docker images that you have built following the Dockerfile manifest on top of the infrastructure you have built following the CloudFormation or Terraform template of your choice.
This definition includes basic elements like networks, volumes, secrets, configs and services.







