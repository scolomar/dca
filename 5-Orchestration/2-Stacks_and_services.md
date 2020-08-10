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






