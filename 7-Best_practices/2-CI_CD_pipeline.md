## Chapter 7: Best practices

### Section 2: CI/CD pipeline

Continuous Integration is about testing the new code as soon as it is developed.
Continuous Delivery is about delivering artifacts ready for production once the release of new code has been validated.
Continuous Deployment is about deploying the new code release into production as soon as it is delivered.

Continuous Deployment is considered to be a more aggressive strategy. 
Many users opt out of this option and prefer to implement Continuous Delivery instead.
In this case the final deployment needs to be manually approved avoiding the automatic deployment of new code to the production environment.

In order to implement a CI/CD pipeline you need to consider the following process:
1. Build an image with the new code.

   Once the new code is developed and committed you need to test it.
   For that purpose you will create an image that will contain the code you want to test together with all the necessary dependencies.
   There are two choices of implementation:
   1. Virtual machines
   1. Docker containers
   
   Both cases are equally effective though Docker containers are proven to spare resources and ease the delivery and maintenance of the whole process.
   Docker containers are typically one order of magnitude less resource expensive than virtual machines.
   Virtual machines are nevertheless considered to be more secure than containers. 
   Following best practices allows nevertheless to safely deploy containerized applications into a production environment.
   The different underlying technologies that support the deployment of containers and virtual machines are the cause for these differences in security and performance.
   
   In order to build the image with the new code you need first to retrieve it.
   It is ideally hosted under a software version control system.
   Git is considered the best option by far because of the many improvements it provides when compared with other competing systems like SVN for example.
   It is highly recommended anyway to use some sort of version controlled repository to easily allow for reverts, blames, merges, tags and many other useful features.
   Using git will for example allow for easy tagging of new releases as well as rollback to a previous version of the code.
   Git is used nowadays by a majority of development teams using different flavours such as Github, Gitlab, Bitbucket and others.
   
   The process of building the image is typically automated by Jenkins, Travis or any other automation tool.
   The event of committing new code to the git repository (or maybe the event of tagging a new release) will trigger a RESTful API call to Jenkins that will build the new image from a manifest: Dockerfile in the case of building Docker images.
   This manifest will contain all the necessary instructions as to build the new image with the new code and all the necessary dependencies.
   
   The new image we want to build will typically contain all the necessary elements that our application needs in order to be successfully deployed.
   This normally includes a basic operating system and all the binaries, libraries and necessary dependencies.
   There is nevertheless an exception to this rule.
   It is obvious that the deployment of our image in the different environments will require different configurations.
   For example the database credentials will not be the same in production than in the development or testing environments.
   The same principle applies for the proxy configuration or even the hardware resources.
   Including this configuration inside the image will make it unable to be deployed in any other environment.
   It will also force us to maintain a different image for each different environment.
   This will also pose the risk that the promotion of an image from the testing environment to production might provoke a failed deployment since they are not identical images.
   The principle of the CI/CD pipeline is that the image that has been automatically tested and promoted for delivery is exactly the same image that is therefore migrated from one environment to the other.
   
   In order to avoid hardcoding our image with the configuration of a specific environment we will not include these data inside the image.
   We will use for this purpose env files, Docker configs, secrets and even external volumes to provision the necessary environment values or configuration files.
   Kubernetes allows for almost exactly the same features with its ConfigMaps, Secrets and Volumes.
   There are only minor differences between the Docker and Kubernetes objects.
   Both of them help us achieve the same target that is to avoid hardcoding the image.
   We will easily migrate the same exact image from one environment to another modifying only the content of the different configuration objects.
   For example the secret containing the database credentials will be different in the testing and the production environments but the image using this secret will be exactly the same.
   This will ensure that the image that has passed the tests and been promoted for delivery is exactly the same image that will be deployed in production.
1. Test the image with the new code.

   Now you need to deploy your application using the image with the new code you want to test.
   In this case you will again use another manifest: the compose file (Docker compose or Kubernetes manifest depending on the orchestrator of your choice).
   The Docker compose will contain all the necessary comp
