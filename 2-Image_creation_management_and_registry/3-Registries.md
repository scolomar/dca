## Chapter 2: Image Creation, Management and Registry

### Section 3: Registries

A Docker registry is a repository that will host your Docker images.
The default Docker registry is Docker Hub at HUB.DOCKER.COM and it hosts the official Docker images for thousands of world-wide software providers.
It is free to use though some extra services might require a paying account.
For me it is more than worth the money.

Of course you can host your own private Docker registry.
This is a common practice for some customers that want maximum control over the host services.
Many customers are though migrating to a sort of managed registry so that it remains private to the company but hosted in the cloud by another party.
I personally think that letting the registry be hosted somewhere in the cloud relieves the operations team from an unnecessary burden.
Each customer will have a different use case that needs to be analized before taking an architectural design decision.

The main advantage of a Docker registry is that you will be able to pull your Docker image from any environment.
You do not need to build your Docker images in the worker nodes.
You will instead use some kind of automation tool like Jenkins to build the Docker image in a build server and then push that image to the Docker registry.
The worker nodes will then download the Docker images from the Docker registry.
The same principle applies for both development and production environments.

It is important to understand that it is not possible for the manager nodes to transfer the Docker images to the worker nodes.
The managers will normally provide the worker nodes with the necessary Docker secrets and configs as well as other metadata and control management data but Docker images are too heavy to be transferred this way.
Each worker node will instead download the Docker images they need for the containers they are running.
In this sense the Docker registry will prove to be very useful as a remote service that will provide the necessary Docker images for the worker nodes.

Typically customers have more than one Docker registry.
It is considered a good practice to have at least one Docker registry for production and a different for development and testing.
The reason behind this approach is to ensure that no unexpected activity in the Docker registry might make it unavailable to the production business.

Enabling the cache will also improve the speed of the download process which might otherwise take too long in certain deployments depending on the size of the Docker images.
It is important to correctly tag the Docker images so as to avoid that the cache will prevent from downloading the correct version of the software.
I have witnessed some cases where the customer was incorrectly using a cached version of the Docker image instead of the new release because of a wrong tag implementation.

Though it is part of a different process I would also like to mention here the importance of checking the Dockerfile template.
If the Dockerfile is poorly designed the build process might reuse cached layers instead of building a brand new Docker image. 
That would therefore result in a Docker container running an outdated version of the software.
