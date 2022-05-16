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

Another important threat to take into consideration is the possibility to modify the content of the Docker image.
Some malicious attacker or just an operator by mistake can modify the content so that what we are going to deploy in our production servers is not what was supposed to be.
To avoid this problem we can cryptographically sign the content of the image in order to verify its integrity.
A simple but very effective approach is to use the digest when referring the Docker images.
The digest is created by a cryptographic function and cannot be modified without being detected.
The safest way to pull an image for our containers is to specify the digest together with the name and location.

Images in Docker Hub are regularly updated to apply security fixes. 
After those fixes are applied the release number is typically not modified.
It happened to one of my customers that they deployed a container in production which depended on an image which was pulled from Docker Hub.
Only the release name was specified in the deployment manifest but not the digest.
As a result, the security fixes that were applied that week to the Docker image made some changes that broke the production deployment of my customer.
The business was stopped for a few hours until they found out the root cause of the problem.
The same deployment manifest that perfectly worked the previous week now failed.
No changes had been made but a new pull caused the download of the modified Docker image but the realease version was identical.
I highly advise to specify the digest in the templates and manifests.
The digest is mathematically impossible to be forged or tampered with.
Any change of a single character in the content of the Docker image would generate a totally different digest.
It will protect not only from malicious attacks but also from mistakenly modifying the content or pointing to a wrong image.

## Exercise

Let us see how we can identify the digest of a Docker image:
```
$ docker inspect tomcat | grep RepoDigests -A2
        "RepoDigests": [
            "tomcat@sha256:aa7b46f45779e295516930c778200b812add66f99595293065bad7030091f6ca"
        ],
```
Once we know the digest we cand specifically download that exact version of the Docker image:
```
docker pull tomcat@sha256:aa7b46f45779e295516930c778200b812add66f99595293065bad7030091f6ca
```

Docker will ignore the image tag and only take into consideration the digest in order to identify the Docker image target.
We can even use a fake tag that it will not affect the download:
```
docker pull tomcat:fake-tag-1.2.3@sha256:aa7b46f45779e295516930c778200b812add66f99595293065bad7030091f6ca
```

This simple but effective strategy will avoid the danger of downloading the wrong Docker image.

Another important task in order to ensure the security of our Docker image is to check its content looking for possible exploits or bugs.
There is plenty of third party solutions that will automatically perform this check otherwise we will need to check it by ourselves.
The process is basically checking the versions of the packages and modules that are installed on your Docker image in order to find security issues.
Inspection of the Dockerfile can provide that information.
