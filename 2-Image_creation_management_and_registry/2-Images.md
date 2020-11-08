## Chapter 2: Image Creation, Management and Registry

### Section 2: Images

Creating a Docker image is the first step when we want to containerize an application.
The second step would be to create a Docker compose to configure the deployment.

To create a Docker image you need first to understand the software.
This means investigating the source code and understanding how it works.
Some useful points to consider would be: 
* Is the application listening in a socket?
* In which port?
* What are the necessary dependencies?
* What is the main program that our application will be running?
* Do we need to compile the code or is it interpreted?
* Is there any necessary parameter or argument for the main command?
* Is the application going to write anything to the filesystem?
* Do we need external volumes?
* What are the necessary environment variables?

The answer to all these questions will help us create a Dockerfile to build a secure and efficient Docker image.

We need to take into account that the Docker image will be the only available filesystem for the container (besides any external volume) so that the software code (compiled or interpreted) together with all the necessary libraries and dependencies should be contained in that image.

For example if we are containerizing a Java application then our Docker image should at least contain the Java Runtime Environment together with all the necessary libraries and dependencies.
By the contrary the Docker image will not need to contain the JAR or WAR file since that could optionally be in an external volume.

In the case of a Python application our Docker image should contain the python binaries together with any requirement or dependencies that our script would need to be run.
The script itself could optionally be located in an external volume.
This same approach would be valid for any other interpreted language like PHP or Ruby.

If our application is written in C/C++ then we will need the compiled binaries together will all the necessary libraries and dependencies otherwise our application will not work.

So that our image should contain the necessary binaries, libraries and dependencies for our application to run.
Nevertheless we do not want to hardcode our images.
We will use external volumes or environment variables to provision any necessary configuration.
For example tokens and passwords will not be included inside the image.
Those elements will vary from one environment to another.
Obviously the database password will be different in the development environment than in production.
But we would like to use the same image in both environments.
For that reason we would not like to include the database password hardcoded inside the image.
Instead we could for example store the password in a Docker secret and provision that secret through an environment variable to the application container.
This way we will use Docker secrets, configs, volumes and environment variables to provision specific configurations to the containers in order to avoid hardcoding our images.
Therefore the same Docker image could be used in production or in the testing environment.
The specific configuration will be provisioned when deploying in a specific environment.

Besides putting apart the configuration of the environment we will try to include as few elements as possible inside the image.
We will avoid including binaries that could potentially be used by a hacker in a compromised container.
If we need to compile our Java code with Maven or our C program with the GNU C Compiler then we will not include those building tools in the final image that will be deployed in production.
The less resources we have available in the production container the more secure it will be because of reducing the attack surface.
Besides improving the container security this measure will also improve the performance of the containerized application since the less amount of content in the container image will improve the performance of the container filesystem.
The key to remember is that fewer and thinner image layers will improve not only the security but also the performance of the final container.