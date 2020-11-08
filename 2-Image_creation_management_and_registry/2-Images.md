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

tbc
