## Chapter 7: Best practices

### Section 2: CI/CD pipeline

Continuous Integration is about testing the new code as soon as it is developed.
Continuous Delivery is about delivering artifacts ready for production once the release of the new code has been validated.
Continuous Deployment is about deploying the new code release into production as soon as it is delivered.

Continuous Deployment is considered to be a more aggressive strategy. 
Many users opt out of this option and prefer to implement Continuous Delivery instead.
In this case the final deployment needs to be manually approved avoiding the automatic deployment of new code to the production environment.

In order to implement a CI/CD pipeline you need to consider the following process:
1. Build an image with the new code and push it to the registry.

   Once the new code is developed and committed you need to test it.
   For that purpose you will create an image that will contain the code you want to test together with all the necessary dependencies.
   There are two choices of implementation:
   1. Virtual machines
   1. Docker containers
   
   Both cases are equally effective though Docker containers are proven to spare resources and ease the delivery and maintenance of the whole process.
   Docker containers are typically one order of magnitude less resource expensive than virtual machines.
   Virtual machines are definitively more secure than containers. 
   Following best practices allows nevertheless to safely deploy containerized applications into a production environment.
   The different underlying technologies that support the deployment of containers and virtual machines are the cause for these differences in security and performance.
   
   In order to build the image with the new code you need first to retrieve it.
   The code will ideally be hosted under a software version control system.
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
1. Test the image.

   Now you need to deploy your application using the image with the new code release you want to test.
   In this case you will again use another manifest: the compose file (Docker compose or Kubernetes manifest depending on the orchestrator of your choice).
   The Docker compose will contain all the necessary objects and definitions for your application to be successfully deployed.
   This typically includes the networks, configs, secrets and volumes as well as any other necessary element.
   
   We will still try to avoid hardcoding the deployment configuration to a specific environment so that our manifest will be as generic as possible.
   For example the description of the secret containing the database credentials will not contain any reference to the environment where we want to deploy our application.
   This way the same Docker compose file will be valid to deploy our application in the testing or in the production environment.
   Although the content of the Docker secret object will obviously be different in both cases.
   
   We also want this Docker compose file to be hosted in a software version control system in order to benefit of the same features such as easy reverting, blaming, merging and tagging.
   It is considered a best practice nowadays to host not only the software but also any operations code or configuration file in a sofware version control system such as git for example.
   This approach is sometimes called GitOps.
   Hosting our Docker compose file in a git repository will make it easy to be retrieved with the absolute certainty regarding the release or commit we have downloaded and are about to test.
   
   The testing process can also be automated through Jenkins, Travis or any other automation tool.
   Jenkins for example will use the Docker compose file to deploy our application and apply all the necessary tests.
   Once the tests have been successfully passed the image will be promoted to the next stage.
   Promoting an image can be as simple as renaming it.
   The content does not actually change at all.
   In some use cases the image will be pushed to another registry exclusive for validated production images.
   Having a separated registry for the production images is considered to be a good practice to ensure the security and performance of the deployments.
   When this is not an option in order to avoid the extra cost of building a separated registry then the digest associated with the tagged image should be enough to ensure a safe deployment.
   We would highly recommend the use of digests to reference the images instead of just relying on the tag.
   The digest is based on a secure cryptographic hash of the image file and guarantees that the production image has not be tampered with or mistakenly retagged.
1. Deploy the image in production.

   Once the image has been promoted and the deployment has been approved we will deploy the new code release in production.
   We will again use a Docker compose manifest (or a Kubernetes manifest if that were our choice).
   Ideally the manifest will exactly be the same with the only minor difference of the version of the release.
   The content of the Docker configs and secrets will provision the adequate configuration data specific to the production environment.

