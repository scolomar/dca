## Chapter 6: Security

### Section 1: Image security

Docker images are immutable. That is a good start for security. 

Though the images are immutable there are still some concerns regarding security.
As you may remember Docker images are the layers of the filesystem that will be used by any application running inside a container.
They contain the libraries, binaries and dependencies for the applications.
And that content will be accesible from inside the container.
We need therefore to avoid any content that might jeopardize the security of the container.

For example it is important not to include in the Docker image any build tool.
Build tools such as compilers can be used to build malicious software in order to infect or compromise the container.
In case we need to use those builders for our image (for example to use Maven to compile our Java application) then we should use intermediate images for that purpose.
The final production image should not contain any build tool.

Regarding the libaries, binaries and dependencies we need to check that all the code included inside the image is up to date. 
It is typical to use Docker images that contain packages which are outdated or lack the necessary security patches.
There are some external tools that regularly check the content of the Docker images and trigger a warning when they find security threats.

Another important threat to take into consideration is the possibility to tamper the content of the Docker image.
Some malicious attacker or just an operator by mistake can modify the content of the Docker image so that what we are going to deploy in our production servers is not what was supposed to be.
To avoid this problem there are some commercial solutions that will even cryptographically sign the content of the image in order to verify its integrity.
A simple but very effective approach is to use the digest when referring the Docker images.
The digest is created by a cryptographic function and cannot be modified without being detected.
The safest way to pull an image for our containers is to specify the digest together with the name and location.

Images in Docker Hub are regularly updated to apply security fixes. 
After those fixes are applied the release number is typically not modified.
Two years ago it happened to one of my customers that they deployed a container in production which depended on an image which was pulled from Docker Hub.
Only the release name was specified in the deployment manifest but not the digest.
As a result, the security fixes that were applied that week to the Docker image made some changes that broke the production deployment of my customer.
The business was stopped for a few hours until we found out the root cause of the problem.
The same deployment manifest that perfectly worked the previous week now failed.
No changes had been made but a new pull caused the download of the modified Docker image but the realease version was identical.
Since then I always specify the digest in the templates and manifests.
The digest is mathematically impossible to be forged or tampered with.
Any change of a single character in the content of the Docker image would cause a totally different digest.
It will protect not only from malicious attacks but also from mistakenly modifying the content or pointing to a wrong image.

The Docker image might contain some metadata that affects the execution of the containers based on that image like for example the user to execute the code or the working directory where to run the application. 
We will discuss those parameters in the next section specifically dedicated to the container security.
