## Chapter 2: Image Creation, Management and Registry

### Section 1: Dockerfile

There are two fundamental manifests in any deployment of a containerized application.
The first one is the Dockerfile that defines how our software is embedded inside a Docker image.
The second one is the Docker compose file that describes how our application is deployed in a container environment.

The Dockerfile is actually a script that contains all the definitions and commands necessary to set up a Docker image that will contain the software that will run in our containers.
The Docker image is absolutely self-contained so that any necessary dependency for our software should be installed during the build process.
The Dockerfile defines this build process step by step.

We will build our Docker images with the command `docker build` that will read the Dockerfile and the build context and send it to the Docker daemon.
One of the most interesting features of using this kind of manifests is that this task can be automated and run by Jenkins for example in a CI/CD pipeline.
Jenkins will for example retrieve the Dockerfile from a remote git repository and launch the build process.
The Dockerfile will be interpreted and executed line by line by the Docker daemon.
There are two types of lines:
* Those which only provide metadata to the Docker image
* Those which actually modify the filesystem

The lines that only introduce metadata will not modify the filesystem contained in the Docker image and therefore will not create or modify any actual content.
The lines of the Dockerfile that modify the filesystem are lines of code that actually create, delete or modify any element of the filesystem.
It can be for example the creation of a new file, the modification of an existing one or the installation or removal of a software package.

Every time the Docker daemon executes one line that modifies the filesystem a new image layer is created.
Instead of actually modifying the original layer the Docker daemon will create a new image layer that will contain the new file in case of creation or the modified file in case of modification.
This is important to understand in order to optimize our Dockerfiles for both security and performance.

If one line for example creates a new file and a second line deletes that file then the original file will not actually be deleted.
One layer will instead contain the newly created file and a second layer will actually have that file deleted in a very similar way on how git repository works.
The main difference when comparing the behaviour of the Docker build process and the git repository is that git will modify the files line by line.
The changes in a Docker image are instead atomic: the whole file will be modified even if only one single line of the file has been modified.
But remember that the original file will still persist in the original layer: a new layer will be created that will contain the modified file.

This can be a security issue since deleting files that contain passwords or any other sensitive data will not prevent from accessing those data diving into the most inner layers of the Docker image.
At the same time it will introduce performance issues because the operating system will need to check the latest version when trying to access any modified file from a multi-layer Docker image.
In order to improve the security of our Docker images we need to take this into account and consider that any data introduced in one layer will persist no matter what we run in subsequent layers.
This means that any content created in one line of our Dockerfile should be removed in that very same line if we want to definitely remove it from the image layer.
Every line in the Dockerfile that modifies the filesystem will create a new different image layer.

This consideration is also important to improve the performance of our containers: we will try to build as few layers as possible given that the existence of many layers will significantly increase the resources usage of our containerized application.
For that purpose we will use as many times as needed the double ampersand symbol of the shell scripting language: `&&`.
It is common to see Dockerfiles that contain dozens or even hundreds of commands chained together in the same line with the double ampersand symbol.

Let us consider one simple example to understand how we can improve the performance of our Docker images.
Let us suppose that we need to install three software packages in our Docker image such as `curl`, `git` and `tcpdump`.
We can for example install these packages in three different lines of our Dockerfile:
```
RUN apt-get --assume-yes install curl
RUN apt-get --assume-yes install git
RUN apt-get --assume-yes install tcpdump
```
But then the result will not be as compact as when installing the three packages in the same line of the Dockerfile like this:
```
RUN apt-get --assume-yes install curl git tcpdump
```
Or even this would be a plausible way:
```
RUN apt-get --assume-yes install curl && apt-get --assume-yes install git && apt-get --assume-yes install tcpdump
```
The reason for this behaviour is that when installing a new package there is a modification of the database of the package manager.
This database is contained in a single file that will be modified three times in case of using three different lines in our Dockerfile.
Every time we modify the database a new version of the same file will be added to the new image layer.
For this reason we will end up with three copies of the database in our final Docker image.
If we run the installation of the three packages in the same line then we will obtain a single modified version of the database in the final Docker image saving this way some space and also reducing the number of image layers.

#### Multi-stage builds

When you build a Docker image you normally need to go through the following process:
1. Fetch the code from the repository
1. Compile the code to get a valid artifact
1. Encapsulate the resulting artifact in a final Docker image

You do not need to keep any of the intermediate results of these steps.
For example after fetching the specific branch or release then you can remove the rest of the repository or even the software used to download the code from the remote repository.
After you have successfully compiled the code and created the final executable binary or Java artifact you can then remove the additional build tools as well as the libraries and dependencies used in the process.
In the final Docker image that you are going to deploy in the production environment you only need the resulting artifact and any necessary dependencies.
This way you are improving the security and performance of your final production image.
The security is improved because you are reducing the attack surface when removing unnecessary software.
The performance is improved because you are reducing the number of layers and total size of the final image.

In a multi-stage build your Dockerfile consists of several FROM instructions.
Each FROM instruction will start the build of an intermediate image.
The result of an intermediate image will be used in the next step removing any dependency used to generate that result.
In the end you will have your small and secure final image ready for a production environment.

Let us see an example of such a multi-stage build:
```
FROM alpine/git AS clone
WORKDIR /app
RUN git clone https://github.com/spring-projects/spring-petclinic.git

FROM maven:alpine AS build
WORKDIR /app
COPY --from=clone /app/spring-petclinic .
RUN mvn install && mv target/spring-petclinic-*.jar target/spring-petclinic.jar

FROM openjdk:jre-alpine AS production
WORKDIR /app
COPY --from=build /app/target/spring-petclinic.jar .
ENTRYPOINT ["java","-jar"]
CMD ["spring-petclinic.jar"]
```

This simple Dockerfile is a perfect example of an optimal build of a Docker image using a Java artifact that will be deployed in production with maximal security and performance.
In the first stage `clone` we fetch the code using `git` but this software will not be available in our production image.
In the second stage `build` we use Maven to build the final artifact using many dependencies that will not appear in the final image.
In our production image we use the resulting artifact of the `build` stage and a Java Runtime Environment to execute our Java application.
This tiny image will be deployed in production maximizing the security and performance of our containerized application.

This is obviously only an example.
Your use case might need more complex stages or even dependencies to run the final artifact in the production environment.

#### Environment variables and arguments

#### .dockerignore
