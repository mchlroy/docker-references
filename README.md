# Most used commands
> Details on how they work below  

`docker pull <image identifier>`  
`docker run -it <image identifier>`   
`docker ps -a`  
`docker stop <CONTAINER ID or NAME>`  
`docker rm <CONTAINER ID or NAME>`  
`docker system prune`
`docker inspect <image identifier>` (must be present locally)  

# Terminology
- *Images* - The file system and configuration of our application which are used to create containers. To find out more about a Docker image, run `docker inspect alpine`. In the demo above, you used the `docker pull` command to download the **alpine** image. When you executed the command `docker run hello-world`, it also did a `docker pull` behind the scenes to download the **hello-world** image.
- *Containers* - Running instances of Docker images &mdash; containers run the actual applications. A container includes an application and all of its dependencies. It shares the kernel with other containers, and runs as an isolated process in user space on the host OS. You created a container using `docker run` which you did using the alpine image that you downloaded. A list of running containers can be seen using the `docker ps` command.
- *Docker daemon* - The background service running on the host that manages building, running and distributing Docker containers.
- *Docker client* - The command line tool that allows the user to interact with the Docker daemon.
- *Docker Store* - A [registry](https://store.docker.com/) of Docker images, where you can find trusted and enterprise ready containers, plugins, and Docker editions. You'll be using this later in this tutorial.  


_[Taken from docker/labs](https://github.com/docker/labs/blob/master/beginner/chapters/alpine.md)_

# Images
## Pulling
Use `docker pull <image identifier>` to download an image onto your disk.  
You can consult the list of images on the disk by using the following command `docker images`.  For more information on an image present on the system, run `docker inspect <image identifier>`.
#### Example
`docker pull alpine`

> If you are using **Docker for Windows**
> `docker4w/nsenter-dockerd` might appear in the images list. This is an image used internally to remotely administrate the Linux VM >(mount shared drives, ...) shipping with Docker for Windows (it is not pulled, but loaded from disk. [Source](https://github.com/docker/for-win/issues/404#issuecomment-274103782)

# Running a container
Using the command `docker run <image identifier>` to run a Docker container based on the image specified  

#### Example
`docker run alpine ls -l`  
Here, we asked Docker to run a container based on the image `alpine` and then executing the command `ls -l` in the container, which will list the computer files on the root of the container.  

  If you run a container based on an image not already pulled, it will pull it behind the scene.

## What happens when you run a container
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulls the image from the Docker Hub (or locally if pulled).
3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.

## Interactive
If you don't want to container to simply boot up, run a command and then kill itself, you should use the `-it` parameter like so: `docker run -it <image identifier>`.
#### Example
`docker run -it alpine /bin/sh` to enter system shell.

## Ports
The `-P` parameter of `docker run` will publish all the exposed container ports to random ports on the Docker host.
To specify the ports forwarding when running, you can use the `-p` command like so: `docker run <image identifier> -p 80:8888` which will forward the host's port 8888 to the container's port 80

`docker port <container's name or id>` will list the ports of container. If you are serving a website for example, going to the address `http://localhost:<PORT FOR 80/tcp show by the docker port command>` 


#### Example
`docker run static-site -P`
```
docker port static-site
443/tcp -> 0.0.0.0:32770
80/tcp -> 0.0.0.0:32771
```
Then going to http://localhost:32771/ will give the website running from a Docker container.

## Showing a list of containers
`docker ps` will list the containers currently running. Using `docker ps -a` will list ALL containers (default shows just running).

## Stopping and removing a running container
Use `docker ps` to find the container's id or name. Then run the following command `docker stop <CONTAINER ID or NAME>` to stop it and `docker rm <CONTAINER ID or NAME>`to remove it from the `docker ps -a` list.
  
> You don't have to type all of the characters of the container's id if what is typed is enough to identify a unique container

> To remove ALL stopped containers at once, run `docker system prune`, which will also remove
> - all networks not used by at least one container
> - all dangling images
> - all dangling build cache

## Useful parameters for `docker run`
- `-e` to pass environment variables. Example: `docker run <image id> -e AUTHOR="Your Name"`
- `--name` to give a name to the container. Example: `docker run <image id> --name "A docker"
- `-P` will publish all the exposed container ports to random ports on the Docker host
- `-d` will create a container with the process detached from our terminal (run in background of the terminal)

# Dockerfile
A Dockerfile is a file that allows you to create images based on an another image. It contains all the informations needed to run an image and commands to execute while creating an image.

## Dockerfile commands
- `FROM` specifies the image to add functionalities to. Required to be the first command.
- `RUN` will run a specified command in the container during creation (used to install dependencies for example). Each run command creates a new layer of the image so that we can rollback easily.
- `COPY` will copy a file from one location on the host (first parameter) to a location in the container (second parameter)
- `EXPOSE` will expose a port from the container to be later mapped on the host machine (appears when inspecting with `docker inspect`). Only used for self documenting purposes.
- `CMD` is the command for running the container/application. Can only appear once.
- `PUSH` command to push the image to Docker Cloud or a private registry.


#### Example Dockerfile for a simple [flask](http://flask.pocoo.org/) app
```
# Base image
FROM alpine:3.5

# Install python and pip
RUN apk add --update py2-pip

# Install Python modules needed by the Python app
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# Copy files required for the app to run
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/

# Tell the port number the container should expose (flask apps defaults to 5000)
EXPOSE 5000

# Run the application
CMD ["python", "/usr/src/app/app.py"]
```

## Building using Dockerfile
`docker build -t <username>/<containername> .` to create an image with `<username>/<containername>` tag and at location `.`.