<a href="Readme.md#table-of-contents" class="top" id="preface">Top</a>
<a id="alpine"></a>
## 1. Playing with Alpine
Now that you have everything setup, it's time to get our hands dirty. In this section, you are going to run a [Alpine Linux](http://www.alpinelinux.org/) container (a lightweight linux distribution) on our system and get a taste of the `docker container run` command.

To get started, let's run the following in our terminal:
```
$ docker image pull alpine
```

> Note: Depending on how you've installed docker on your system, you might see a `permission denied` error after running the above command. If you're on a Mac, [verify your installation](https://docs.docker.com/mac/step_one/). If you're on Linux, you may need to prefix your `docker` commands with `sudo`. Alternatively you can [create a docker group](http://docs.docker.com/engine/installation/ubuntulinux/#create-a-docker-group) to get rid of this issue.

The `pull` command fetches the alpine **image** from the **Docker registry** and saves it in our system. You can use the `docker image ls` command to see a list of all images on your system.
```
$ docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
alpine                 latest              c51f86c28340        4 weeks ago         1.109 MB
hello-world             latest              690ed74de00f        5 months ago        960 B
```

<a id="dockerrun"></a>
### 1.1 Docker Run
Great! Let's now run a Docker **container** based on this image. To do that you are going to use the `docker container run` command.

```
$ docker container run alpine ls -l
total 48
drwxr-xr-x    2 root     root          4096 Mar  2 16:20 bin
drwxr-xr-x    5 root     root           360 Mar 18 09:47 dev
drwxr-xr-x   13 root     root          4096 Mar 18 09:47 etc
drwxr-xr-x    2 root     root          4096 Mar  2 16:20 home
drwxr-xr-x    5 root     root          4096 Mar  2 16:20 lib
......
......
```
What happened? Behind the scenes, a lot of stuff happened. When you call `run`, the Docker client finds the image (alpine in this case), creates the container and then runs a command in that container. When you run `docker container run alpine`, you provided a command (`ls -l`), so Docker started the command specified and you saw the listing.

Let's try something more exciting.

```
$ docker container run alpine echo "hello from alpine"
hello from alpine
```
OK, that's some actual output. In this case, the Docker client dutifully ran the `echo` command in our alpine container and then exited it. If you've noticed, all of that happened pretty quickly. Imagine booting up a virtual machine, running a command and then killing it. Now you know why they say containers are fast!

Try another command.
```
$ docker container run alpine /bin/sh
```

Wait, nothing happened! Is that a bug? Well, no. These interactive shells will exit after running any scripted commands, unless they are run in an interactive terminal - so for this example to not exit, you need to `docker container run -it alpine /bin/sh` which will be discussed later on.

Ok, now it's time to see the `docker container ls` command. The `docker container ls` command shows you all containers that are currently running.

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Since no containers are running, you see a blank line. Let's try a more useful variant: `docker container ls -a`

```
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
36171a5da744        alpine              "/bin/sh"                5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
305297d7a235        alpine             "uptime"                  5 minutes ago       Exited (0) 4 minutes ago                        distracted_goldstine
a6a9d46d0b2f        alpine             "echo 'hello from alp"    6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
ff0a5c3750b9        alpine             "ls -l"                   8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock
```

What you see above is a list of all containers that you ran. Notice that the `STATUS` column shows that these containers exited a few minutes ago. You're probably wondering if there is a way to run more than just one command in a container. Let's try that now:

```
$ docker container run -it alpine /bin/sh
/ # ls
bin      dev      etc      home     lib      linuxrc  media    mnt      proc     root     run      sbin     sys      tmp      usr      var
/ # uptime
 05:45:21 up  5:58,  0 users,  load average: 0.00, 0.01, 0.04
```
Running the `run` command with the `-it` flags attaches us to an interactive tty in the container. Now you can run as many commands in the container as you want. Take some time to run your favorite commands like `ls -l` or `uptime`. Exit the interactive shell via the `exit`command.

That concludes a whirlwind tour of the `docker container run` command which would most likely be the command you'll use most often. It makes sense to spend some time getting comfortable with it. To find out more about `run`, use `docker container run --help` to see a list of all flags it supports. As you proceed further, we'll see a few more variants of `docker container run`.

<a id="terminology"></a>
### 1.2 Terminology
In the last section, you saw a lot of Docker-specific jargon which might be confusing to some. So before you go further, let's clarify some terminology that is used frequently in the Docker ecosystem.

- *Images* - The Filesystem and configuration of our application which are used to create containers. To find out more about a Docker image, run `docker image inspect alpine`. In the demo above, you used the `docker image pull` command to download the **alpine** image. When you executed the command `docker container run hello-world`, it also did a `docker image pull` behind the scenes to download the **hello-world** image.
- *Containers* - Created using Docker images and run the actual application. You created a container using `docker container run` which you did using the alpine image that you downloaded. A list of running containers can be seen using the `docker container ls` command.
- *Docker daemon* - The background service running on the host that manages building, running and distributing Docker containers.
- *Docker client* - The command line tool that allows the user to interact with the Docker daemon.
- *Docker Hub* - A [registry](https://hub.docker.com/explore/) of Docker images. You can think of the registry as a directory of all available Docker images. You'll be using this later in this tutorial.

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="webapps"></a>
## 2. Webapps with Docker
Great! So you have now looked at `docker container run`, played with a docker container and also got a hang of some terminology. Armed with all this knowledge, you are now ready to get to the real-stuff i.e. deploying web applications with Docker.

<a id="static-site"></a>
### 2.1 Static Sites
Let's start by taking baby-steps. The first thing we're going to look at is how you can run a dead-simple static website. You're going to pull a docker image from the docker hub, run the container and see how easy it so to set up a webserver.

The image that you are going to use is a single-page website that was already created for this demo and is available on the Docker Hub as [`seqvence/static-site`](https://hub.docker.com/r/seqvence/static-site/). You can download and run the image directly in one go using `docker container run`.

```
$ docker container run seqvence/static-site
```
Since the image doesn't exist on your Docker host, the Docker daemon will first fetch the image from the registry and then run the image.
Okay, now that the server is running, do you see the website? What port is it running on? And more importantly, how do you access the container directly from our host machine?

In this case, the client didn't tell the Docker Engine to publish any of the ports so you need to re-run the `docker container run` command. We'll take the oportunity to publish ports and pass your name to the container to customize the message displayed. While we are at it, you should also find a way so that our terminal is not attached to the running container. So that you can happily close your terminal and keep the container running. This is called the **detached** mode.

Before we look at the **detached** mode, we should first find out a way to stop the container that you have just launched.

First up, launch another terminal (command window) and execute the following command. If you're using docker-machine you need to run `eval $(docker-machine env <YOUR_DOCKER_MACHINE_NAME>)` in each new terminal otherwise you'll get the error "Cannot connect to the Docker daemon. Is the docker daemon running on this host?".
```
$ docker container ls
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
a7a0e504ca3e        seqvence/static-site   "/bin/sh -c 'cd /usr/"   28 seconds ago      Up 26 seconds       80/tcp, 443/tcp     stupefied_mahavira
```

Check out the `CONTAINER ID` column. You will need to use this `CONTAINER ID` value, a long sequence of characters and first stop the running container and then remove the running container as given below. The example below provides the `CONTAINER ID` on our system, you should use the value that you see in your terminal.
```
$ docker container stop a7a0e504ca3e
$ docker container rm   a7a0e504ca3e
```

Note: A cool feature is that you do not need to specify the entire `CONTAINER ID`. You can just specify a few starting characters and if it is unique among all the containers that you have launched, the Docker client will intelligently pick it up.

Now, let us launch a container in **detached** mode as shown below:

```
$ docker container run --name static-site -e AUTHOR="Your Name" -d -p 8080:80 seqvence/static-site
e61d12292d69556eabe2a44c16cbd54486b2527e2ce4f95438e504afb7b02810
```

In the above command, `-d` will create a container with the process detached from our terminal, `-p 8080:80` will publish the exposed container port 80 to the 8080 port on the Docker host, `-e` is how you pass environment variables to the container, and finally `--name` allows you to specify a container name. `AUTHOR` is the environment variable name and `Your Name` is the value that you can pass.

You can now open [http://127.0.0.1:8080](http://127.0.01:8080) to see your site live! 
_Hint: In AWS Cloud9 use `Tools -> Preview -> Preview running applications` to open the browser on the appropriate remote address._

<img src="https://raw.githubusercontent.com/docker/Docker-Birthday-3/master/tutorial-images/static.png" title="static">

I'm sure you agree that was super simple. To deploy this on a real server you would just need to install docker, and run the above docker command.

Now that you've seen how to run a webserver inside a docker image, you must be wondering - how do I create my own docker image? This is the question we'll be exploring in the next section. But first, let's stop and remove the containers since you won't be using them anymore.

```
$ docker container stop static-site
$ docker container rm static-site
```

<a id="docker-images"></a>
### 2.2 Docker Images

You've looked at images before but in this section we'll dive deeper into what docker images are and build our own image. And, we'll also use that image to run our application locally. Finally, you'll push some of your images to Docker Hub.

Docker images are the basis of containers. In the previous example, you **pulled** the *seqvence/static-site* image from the registry and asked the docker client to run a container **based** on that image. To see the list of images that are available locally, use the `docker image ls` command.

```
$ docker image ls
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
seqvence/static-site   latest              92a386b6e686        2 hours ago        190.5 MB
nginx                  latest              af4b3d7d5401        3 hours ago        190.5 MB
python                 2.7                 1c32174fd534        14 hours ago        676.8 MB
postgres               9.4                 88d845ac7a88        14 hours ago        263.6 MB
containous/traefik     latest              27b4e0c6b2fd        4 days ago          20.75 MB
node                   0.10                42426a5cba5f        6 days ago          633.7 MB
redis                  latest              4f5f397d4b7c        7 days ago          177.5 MB
mongo                  latest              467eb21035a8        7 days ago          309.7 MB
alpine                 3.3                 70c557e50ed6        8 days ago          4.794 MB
java                   7                   21f6ce84e43c        8 days ago          587.7 MB
```

The above gives a list of images that I've pulled from the registry and the ones that I've created myself (we'll shortly see how). The list will most likely not correspond to the list of images that you have currently on your machine. The `TAG` refers to a particular snapshot of the image and the `ID` is the corresponding unique identifier for that image.

For simplicity, you can think of an image akin to a git repository - images can be [committed](https://docs.docker.com/engine/reference/commandline/commit/) with changes and have multiple versions. When you do not provide a specific version number, the client defaults to `latest`.

For example, you can pull a specific version of `ubuntu` image as follows:

```
$ docker image pull ubuntu:12.04
```

**NOTE**: Do not execute the above command. It is only for your reference.

If you do not specify the version number of the image, then as mentioned the Docker client will default to a version named `latest`.

So for example, the `docker image pull` command given below will pull an image named `ubuntu:latest`:

```
$ docker image pull ubuntu
```

To get a new Docker image you can either get it from a registry (such as the docker hub) or create your own. There are tens of thousands of images available on [Docker hub](https://hub.docker.com). You can also search for images directly from the command line using `docker search`.

An important distinction to be aware of when it comes to images is between base and child images.

- **Base images** are images that has no parent image, usually images with an OS like ubuntu, alpine or debian.

- **Child images** are images that build on base images and add additional functionality.

Then there are two more types of images that can be both base and child images, they are official and user images.

- **Official images** Docker, Inc. sponsors a dedicated team that is responsible for reviewing and publishing all Official Repositories content. This team works in collaboration with upstream software maintainers, security experts, and the broader Docker community. These are not prefixed by an organization or user name. In the list of images above, the `python`, `node`, `alpine` and `nginx` images are official (base) images. To find out more about them, check out the [Official Images Documentation](https://docs.docker.com/docker-hub/official_repos/).

- **User images** are images created and shared by users like you. They build on base images and add additional functionality. Typically these are formatted as `user/image-name`. The `user` value in the image name is your Docker Hub user or organization name.

Great! We're done with this section. Let's move on to explore the capabilities of Dockerfiles.
<a href="2_dockerfile.md">Goto next section.</a>

<a href="Readme.md#table-of-contents" class="top" id="preface">Top</a>
