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

The Dockerfile will be the key to create and build our image where we will specify the final content of the image as well as the content and number of layers.
If there is any listening port in our application it will also be specified in the Dockerfile as well as any specific user or external volume that needs to be used.

On the other hand it will be the Docker compose file the tool to configure the deployment of our image. 
There we will specify arguments, volume configurations, Docker secrets and configs, environment variables as well as any other circumstance that we want to include in the deployment of our containerized application.
If we were using Kubernetes instead of Docker Swarm then we would use a similar Kubernetes compose file to describe the deployment.
The basic difference between a Docker compose file and a Kubernetes compose file is that Docker compose is a unique file that describes the whole deployment meanwhile Kubernetes uses a set of configuration files that will independently describe different elements such as the network policy, the service and the deployment itself for example.
All these different Kubernetes components are tied together through the use of labels.

Those two files (Dockerfile and Docker compose file) are the keys to successfully containerize any software application.

## Exercise

Let us suppose that we want to deploy on a container platform an application written in Python like this one:
```
#!/usr/bin/env python3
import collections
fichero='/data/words.txt'
lista=sorted(list(set([palabra.strip().lower() for palabra in open(fichero,'r')])))
def unico(palabra):
  return ''.join(sorted(palabra))
conjunto_unicos=collections.defaultdict(list)
for palabra in lista:
  conjunto_unicos[unico(palabra)].append(palabra)
def anagramas(palabra):
  return conjunto_unicos[unico(palabra)]
def anagramas_lista(lista):
  return {palabra:anagramas(palabra) for palabra in lista if len(anagramas(palabra))>1}
def anagramas_lista_cuenta(lista):
  return {palabra:len(anagramas(palabra))-1 for palabra in lista if len(anagramas(palabra))>1}
def anagramas_lista_max(lista):
  cuenta_max=0
  for palabra in lista:
    if len(anagramas(palabra))-1>cuenta_max:
      cuenta_max=len(anagramas(palabra))-1
      palabra_max=palabra
  return {palabra_max: cuenta_max}  
conjunto_tamano=collections.defaultdict(list)
for palabra in lista:
  conjunto_tamano[len(palabra)].append(palabra)
cuenta_anagramas={}
for tamano,palabras in conjunto_tamano.items():
  cuenta_anagramas[tamano]=sum(1 for palabra in palabras if len(anagramas(palabra))>1)
RESULTADO=cuenta_anagramas
print(RESULTADO)
```
This sample application will read a file containing a dictionary, separate words into classes of words with the same length and print the total number of anagrams in each class.

An anagram is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.
For example LISTEN = SILENT.
Please check the reference in Wikipedia:
- https://en.wikipedia.org/wiki/Anagram

If we want to deploy such application in a container we will need to use a Docker image that will contain all the necessary libraries.
We can create that image or we can use an existing image.
Docker Hub is an excellent place to search for an existing Docker image. 
Here we have a Docker image that contains all the necessary Python libraries:
- https://hub.docker.com/_/python

There are many different images available created with different base operating systems. 
Many customers choose Docker images based on Linux Alpine because of their small size.
Here is the command you can use to download this image:
```
docker pull python:alpine
```

And this is the Dockerfile that was used to create this image:
- https://github.com/docker-library/python/blob/master/3.10/alpine3.15/Dockerfile

Let us suppose that you clone this Github repository that contains a sample of dictionary and the Python script:
- https://github.com/academiaonline-org/anagrams

You can clone the repositoy with the following command:
```
git clone https://github.com/academiaonline-org/anagrams && cd anagrams
```

In order to run the container this would be the command:
```
docker run --entrypoint python3 --volume ${PWD}/data/words.txt:/data/words.txt:ro --volume ${PWD}/src/anagrams.py:/data/anagrams.py:ro --workdir /data/ python:alpine anagrams.py
```

Let us explain this command line:
- ```docker run``` will create a container and run our application defined in the entrypoint: ```python3```
- The argument of the entrypoint is written at the end of the line: ```anagrams.py```
- Entrypoint and arguments will be equivalent to directly run on the command line: ```python3 anagrams.py```
- In order for the application to locate the script we need to run our application in the specific working directory defined with the parameter: ```workdir```
- We also need to mount our dictionary file on the container filesystem. We use the option ```volume``` for this purpose.
- We also need to mount the Python script on the container filesystem. We use again the option ```volume``` for this purpose.
- In order to increase the security of our containers we mount these two files in read-only mode adding the option ```:ro``` to the right of the volume option.

The output of the command will be a map of values representing the length of the classes and the number of anagrams in each class.
In my case this is the values that I get:
```
{1: 0, 2: 80, 3: 805, 5: 4497, 4: 2790, 8: 4821, 7: 5759, 9: 3552, 6: 6246, 11: 1054, 10: 2082, 12: 558, 14: 140, 16: 70, 15: 90, 20: 6, 19: 14, 17: 44, 13: 250, 18: 20, 21: 8, 22: 4, 23: 0, 24: 0}
```
The maximum number of anagrams (6246) is achieved with words of 6 characters of length.

This is a very simple example written in Python but you could also use a more complex application designed for data analysis.
