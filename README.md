<a id="top"></a>
<img src="https://raw.githubusercontent.com/docker/Docker-Birthday-3/master/tutorial-images/logo.png" alt="docker logo">
Special thanks to [Docker](https://github.com/docker/docker-birthday-3)  and [Prakhar Srivastav](http://prakhar.me) for parts of this tutorial.

<a href="#top" class="top" id="getting-started">Top</a>
## Getting Started: FAQs

### What is Docker Engine?

Docker Engine is a tool that allows developers, sys-admins etc. to easily deploy their applications in a sandbox (called *containers*) to run on the host operating system i.e. Linux. The key benefit of Docker Engine is that it allows users to **package an application with all of its dependencies into a standardized unit** for software development. Unlike virtual machines, containers do not have the overhead of a full operating system and hence enable more efficient usage of the underlying system and resources.


### What are containers?

The industry standard today is to use Virtual Machines (VMs) to run software applications. VMs run applications inside a guest Operating System, which runs on virtual hardware powered by the server’s host OS.

VMs are great at providing full operating system isolation for applications: there are very few ways a problem in the host operating system can affect the software running in the guest operating system, and vice-versa. But this isolation comes at great cost — the computational overhead spent virtualizing hardware for a guest OS to use is substantial.

Containers take a different approach: by leveraging the low-level mechanics of the host operating system, containers provide most of the isolation of virtual machines at a fraction of the computing power.

### What will this tutorial teach me?
This tutorial aims to be the one-stop shop for getting your hands dirty with Docker. Apart from demystifying the Docker landscape, it'll give you hands-on experience with building and deploying your own webapps. You'll quickly build a multi-container wordpress service. Even if you have no prior experience with deployments, this tutorial should be all you need to get started.

## Using this Document
This document contains a series of several sections, each of which explains a particular aspect of Docker. In each section, you will be typing commands (or writing code). All the code used in the tutorial is available in the [Github repo](https://github.com/uniba-dsg/docker-tutorial/).

<a href="#top" class="top" id="table-of-contents">Top</a>
## Table of Contents

- [Preface](#preface)
    - [Setting up your computer](#setup)
-   [1. Playing with Alpine](1_engine.md#alpine)
    -   [1.1 Docker Run](1_engine.md#dockerrun)
    -   [1.2 Terminology](1_engine.md#terminology)
-   [2. Webapps with Docker](1_engine.md#webapps)
    -   [2.1 Static Sites](1_engine.md#static-site)
    -   [2.2 Docker Images](1_engine.md#docker-images)
    -   [2.3 Our First Image](2_dockerfile.md#our-image)
    -   [2.4 Dockerfile](2_dockerfile.md#dockerfiles)
-   [3. Docker Compose](3_compose.md#compose)
    -   [3.1 Step 1: Setup](3_compose.md#setup)
    -   [3.2 Step 2: Create a Dockerfile](3_compose.md#dockerfile)
    -   [3.3 Step 3: Define services in a Compose file](3_compose.md#composefile)
    -   [3.4 Step 4: Build and run your app with Compose](3_compose.md#ship)
    -   [3.5 Step 5: Edit the compose file to add a bind mount](3_compose.md#mount)
    -   [3.6 Step 6: Re-build and run the app with Compose](3_compose.md#rebuild)
    -   [3.7 Step 7: Update the application](3_compose.md#update)
    -   [3.8 Step 8: Experiment with some other commands](3_compose.md#experiment)
-   [4. Docker Compose and Wordpress](4_compose.md#compose)
    -   [4.1 Define the project](4_compose.md#project)
    -   [4.2 Build the project](4_compose.md#build)
    -   [4.3 Bring up WordPress in a web browser](4_compose.md#run)


<a href="#table-of-contents" class="top" id="preface">Top</a>
## Preface

> Note: This tutorial is based on version **1.10.1** of Docker and up.

<a id="setup"></a>
### Setting up your computer
Getting all the tooling setup on your computer can be a daunting task, but thankfully getting Docker up and running on your favorite OS has become very easy.

The *getting started* guide on Docker has detailed instructions for setting up Docker on [Mac](http://docs.docker.com/mac/step_one/), [Linux](http://docs.docker.com/linux/step_one/) and [Windows](http://docs.docker.com/windows/step_one/).

Once you are done installing Docker, test your Docker installation by running the following:
```
$ docker container run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
03f4658f8b78: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:8be990ef2aeb16dbcb9271ddfe2610fa6658d13f6dfb8bc72074cc1ca36966a7
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.
...
```

<a id="references"></a>
## References
- [What is Docker](https://www.docker.com/what-docker)
- [Docker Cheatsheet](https://github.com/wsargent/docker-cheat-sheet)
- [Docker Compose](https://docs.docker.com/compose)
