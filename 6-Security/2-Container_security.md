## Chapter 6: Security

### Section 2: Container security

Docker containers are not immutable. They are writable by default. That is not a good start for security. 

We can improve this condition by modifying the default behaviour of our containers.
We can force a read-only option for the running containers. This way the container will not be writable at all.
In case of a compromised container the malicious attacker will not be able to modify the content.
This simple feature will dramatically improve the security of our containers.
The performance will also benefit from this approach since we will avoid the superposition of the filesystem layers which affects both the container and image layers.

You may nevertheless ask: how is it possible to make a container non-writable? Normally almost any application needs to write something to the filesystem sooner or later.
How do we handle that situation?

In principle we have the Docker images that are immutable and the Docker containers that are writable by default.
How do our applications write to the filesystem if we make the container also immutable?
The answer to this question is the Docker volume.

We can think of the Docker volume as an external filesystem independent from both the container and the Docker image.
It will be mounted inside the container so that any running application might have access to this filesystem.

Another advantage of using Docker volumes is that it will bypass the Copy-on-Write mechanism avoiding this way the expensive superposition of the container and image filesystem layers.
We will have with this approach three different sources for our filesystem: the immutable image, the container (in read-only mode) and the volume (which can be configure in both read-only or writable mode).

If there is not an actual need to modify the content of the volume it is advisable to mount it in read-only mode since it is a point of access to the container filesystem and therefore a way to penetrate it.
Moreover the volume might be shared by many other containers so that a compromised container with writable access to that volume would be able to infect the neighbour containers through the shared filesystem.

Another important concern regarding the container security is the user that owns the application running inside the container.
Normally it is not recommended to run any application inside a container as root.
It is true that the root user inside a container has limited capabilities but it is considered a good practice to run as non-root if possible.
The user 'nobody' is the ideal candidate to run our applications inside the container though that is not always possible due to permissions and other use cases.

We need to take into consideration that despite the high level of security of the current containerization technology a container can effectively be compromised.
In that case we want to ensure that the isolation of the compromised container will prevent a horizontal escalation of the attack.
The immutability of both containers and Docker images will definitely help for the purpose of isolating the attacker and minimizing the damage.
If the volumes are also in read-only mode then the attack will be practically contained.

## Exercise

Let us run a sample application in a container and let us try to make it as safe as possible:
```
git clone https://github.com/academiaonline-org/anagrams && cd anagrams
docker run --entrypoint python --network none --read-only --volume ${PWD}/data/words.txt:/data/words.txt:ro --volume ${PWD}/src/anagrams.py:/data/anagrams.py:ro --workdir /data/ python anagrams.py
```
Let us examine these commands step by step: the first line will just download the respective Github repository which contains the Python script as well as a sample dictionary.

The second line will actually run the containerized application.
Let us examine the different options of the command line:
- `entrypoint` will specify the command that we want to run as the main process (PID 1) inside the container. In this example we are running the Python interpreter.
- `network` will connect the container to the specified network. In this case we are using `none` as the network so that the container will have no access at all to any external network (only the local loopback network `localhost`). This is not always possible since many containers need external connectivity. Please apply this configuration whenever possible in order to properly isolate the container.
- `read-only` will mount the container's root filesystem as read only. This is a good measure of security for our containers. If it gets compromised it will be impossible to modify the root filesystem as it is mounted as read-only. I would recommend to use this option whenever possible.
- `volume` will mount an external volume inside the container. That is normally necessary when we need to inject a configuration file inside the container or when the running application needs to perform changes to the filesystem, for example. This is useful but it poses serious security threats. In order to minimize the risks I would highly encourage to use the `:ro` flag at the end of the option in order to mount the volume as read-only. Please do so whenever possible. In this example we are using two different volumes: the first one contains the sample dictionary (`words.txt`) and the second one contains the Python script that we want to execute (`anagrams.py`).
- `workdir` is the working directory inside the container for the running application. It is a good idea to use a custom working directory instead of the default (root /).
