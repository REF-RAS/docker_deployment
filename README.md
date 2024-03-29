# Docker Robotic Environment Deployment Package
![QUT REF Collection](https://badgen.net/badge/collections/QUT%20REF-RAS?icon=github) [![License](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

This repo contains files for docker-based setup of system environments. The types of system environments are relevant for project development work of the Robotics and Autonomous Systems (RAS) group and the QUT Centre for Robotics (QCR).

The setup files compose of specifications of layered entities in the docker/docker compose framework, namely, Dockerfiles and Docker-Compose (yaml) files. 

* A Dockerfile specifies the scripts for building a docker image, which is a template of executable system environments. 
* A Docker-Compose (yaml) file specifies the commands for running one or more docker containers based on a docker image. Each executing instance is known as a service. Optionally it comprises the specifications for building docker images, connecting to prescribed system services, disk volumes, and other system entities.

## List of Supported Docker Images

Currently the image specifications mainly emerge from development of robotic applications with ROS. More docker image specifications will be added to this repo.

### Base Images

| Images        | Remarks                           | Depends on    | Ready | Tested Host OS
| ------------- | --------------------------------- | ------------- | ----- | -------------- |
| `rosbase`     | ROS 1 (Noetic) + Ubuntu 20.04     | Nil           | YES   | Ubuntu 20.04
| `ros2base`    | ROS 2 (Humble) + Ubuntu 22.04     | Nil           | YES   | Ubuntu 20.04
| `rosbase-gpu` | ROS 1 (Noetic) + Nvidia           | `rosbase`     | NO    |   Nil
| `moveit`      | ROS 1 (Noetic) + Moveit version 1 | `rosbase`     | YES   | Ubuntu 20.04
| `moveit2`     | ROS 2 (Humble) + Moveit version 2 | `ros2base`    | YES   | Ubuntu 20.04
| `nvidia`      | Ubuntu 20.04 with Nvidia drivers  | Nil           | NO    |   Nil
| `armer`       | ROS 1 (Noetic)+ Armer             | `rosbase`     | NO    |   Nil

### Robot Arm Images

| Images                    | Remarks                                               | Depends on                | Ready | Tested Host OS
| ------------------------- | ----------------------------------------------------- | ------------------------- | ----- | ------------ |
| `abb_humble_moveit`       | ROS 2 (Humble) + Ubuntu 22.04 + ABB base packages     | `moveit2` and `ros2base`  | NO    | Ubuntu 20.04
| `panda_noetic_moveit`     | ROS 1 (Noetic) + Ubuntu 20.04 + Panda prepped space   | `moveit` and `rosbase`    | NO    | Ubuntu 20.04
| `panda_humble_moveit`     | ROS 2 (Humble) + Ubuntu 22.04 + Panda prepped space   | `moveit2` and `ros2base`  | YES   | Ubuntu 20.04
| `xarm_noetic_moveit`      | ROS 1 (Noetic) + Ubuntu 20.04 + xArm prepped space    | `moveit` and `rosbase`    | YES   | Ubuntu 20.04
| `xarm_humble_moveit`      | ROS 2 (Humble) + Ubuntu 22.04 + xArm prepped space    | `moveit2` and `ros2base`  | YES   | Ubuntu 20.04

## File Organization

The main entry point is the __docker-compose.yaml__, with a summarised structure displayed below:

```bash
├── docker_deployment
|   ├── docker-compose.yaml                 # Main entry call through docker compose
|   ├── base-compose.yaml                   # Entry level defining base images (listed below)
|   ├── robot-arm-compose.yaml              # Entry level defining robot arm packages as <ROBOT>_<ROSVER>_<COMMANDER>
|   ├── .env_template                       # An envionment template (used as .env) that defines <ROBOT>, <ROSVER>, and <COMMANDER>
|   ├── devel                               # Folder containing files for demonstrating container-based development in Visual Studio Code
|   ├── docs                                # Documentation folder (reference materials)
|   ├── public                              # Folder containing public items (i.e., pictures, etc.)
|   ├── images
|   |   ├── robot_arms                          # Folder containing all the robot arm images. 
|   |   |   ├── <ROBOT>_<ROSVER>_<COMMANDER>    # Multiple available packages in this format
|   |   ├── base                                # Folder containing all base images.
|   |   |   ├── rosbase
|   |   |   ├── ros2base
|   |   |   ├── moveit
|   |   |   ├── moveit2
|   |   |   ├── armer
```

### The file: docker-compose.yaml 

This file contains commands for starting up docker services using the docker compose command suite. A docker service is an executing container cloned from an image. The file `docker-compose.yaml` is handy for orchestrating one or more services that work together for a project, in addition, specifies supporting components of the services and parameters pertinant to the environment.

The file has a tree structure, and the top level branches to different services. Each service sub-branch contains various information on how to start the service, such as the image, the devices used including the input/output, network, and GPU, the volumes to mount in the container's file system. 

[Compose specification with the compose file](https://docs.docker.com/compose/compose-file/03-compose-file/)

### The directories under `images`

Every directory under `images` contains files relevant for building one image or a family of similar images. They share a similar structure and key entities including the followings.

* Dockerfile: a layer-by-layer list of commands for building a system environment as an image. 
* The 'assets' folder: files to be added to the image as specified in the Dockerfile.
- The `entrypoint_setup.sh`: the script file specified in a Dockerfile that initializes the container (system environment).

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## Pre-requisites

To exploit docker images in the platform setup for a project, simply start with a base OS platform (Linux, Windows, or MacOS) installed with the suite of Docker Engine, Docker CLI and Docker Compose. 

Refer to the [Docker installation page](./docs/DOCKER_INSTALL.md) provided in this repo.

Once the installation is successfully completed, the docker engine should be running in the background (as a service or daemon).

## Building and Running Images

Image building and running are supported in the vanilla docker engine and also the docker compose tools. The latter is easier to use if the commands and parameters have been specified in a docker-compose yaml file.

### Building images using docker-compose

The following commands build the `rosbase` and `armer` images. Note that `armer` has specified `rosbase` as the base image in the Dockerfile. The `rosbase` must be built first. 

```
docker compose build rosbase
docker compose build armer
```
![Layered Docker Image for armer](./public/ArmerDockerLayers.png)

There are at least two ways to create a new container (an instance of executing image) and execute it. Note that executing docker compose in the folder that contains the docker compose yaml file.

Using docker compose up
```
docker compose up armer
```
Using docker compose run (to start an interactive shell). The `-it` flags refer to the `tty` and `stdin_open` settings in the docker compose yaml file. 
```
docker compose run -it armer
```
Note that the `armer` service extends the `rosbase` service and therefore it inherits the settings of the base service.

Use docker compose down to terminate a container.
```
docker compose down
```
The terminated container goes into a dormant state, which can be later restarted with all the environment elements intact. The following lists the containers and their information.
 ```
 docker container ls
 ```
To execute commands on a currently running container, use docker compose exec. The following command starts a bash shell in the currently running container called `armer`.  
```
docker compose exec armer bash
```
Note that docker compose commands must be executed in a folder where the corresponding `docker-compose.yaml` file is located.  If the current directory is somewhere else, use the `-f` or `--file` option to specify the location of the yaml file.  Multiple configuration yaml files may be included.

```
docker compose -f ../compose.yaml exec moveit bash
```

In docker the images are managed by the Docker Engine which can be accessed anywhere with the Docker CLI. An alternative command for executing a bash command in the `moveit` container is given below.

```
docker exec -it <CONTAINER NAME> bash
```
The next section gives a brief comparision of the Docker CLI and the Docker Compose CLI.

You can also add to the docker-compose.yaml file using environment parameters (stored in a .env) file. This can make switching between container services flexible with respect to development and which volumes you wish to mount into the container (for example, project specific paths). please refer to [Environment Compose Pipeline](./docs/COMPOSE_PIPELINE.md) for more information.

Refer to [Docker Compose CLI reference](https://docs.docker.com/compose/reference/) for details of all available docker compose commands.

### Docker CLI and Docker Compose CLI

Docker compose is a plugin or a tool on top of the __origin__ Docker Engine. The __origin__ Docker CLI commands are useful for managing images and containers.  For example to list all containers, use the following
```
docker container ls
```
To remove all stopped containers, use the following. Beware that removing a container means that it cannot be restarted and any change made to the container will be lost.
```
docker container prune
``` 

Note that the compose function only works within the local directory (where the docker compose yaml has been specified). If a container has been created for a composed image, simply use the default Docker commands (i.e., docker exec or run) to interact with the running container (if outside the context of the compose folder)

Refer to the [Docker CLI reference documentation](https://docs.docker.com/engine/reference/commandline/container_ls/) for all the available commands.

## Comparision between Docker Compose YAML and Dockerfile

Essentially, Docker Compose makes running images (as containers) easier. An example based on the `rosbase` image (ROS 1) is given below. First build the `rosbase` image using a Dockerfile in the same folder.
```
docker build -f Dockerfile -t rosbase .
```

The following command will run the `rosbase` image (assumed that the image has already been built)
```
docker run rosbase
```
The container will function but some critical features are missing such as network capability, file volume mounts, locale and graphics. The following will enable the sharing of the host networking capability, the timezone/locale settings, X11, etc with the container. 
```
docker run -it --net=host \
    --env="NVIDIA_DRIVER_CAPABILITIES=all" \
    --env="DISPLAY" \
    --env="QT_X11_NO_MITSHM=1" \
    --env="ROS_MASTER=true" \
    --env="ROS_MASTER_URI=http://localhost:11311"
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    --volume="/etc/timezone:/etc/timezone:ro" \
    --volume="/etc/localtime:/etc/localtime:ro" \
    rosbase \
    bash
```
The troublesome long command is not desirable. This is where Docker Compose can come to the rescue. The command can be transformed as a Docker Compose service and specified in a docker compose yaml file. The following contains the configurations in the above long command and more.
```
version: '3'
services:
    rosbase:
        build:
            context: ./docker/rosbase
            dockerfile: Dockerfile
            args: 
                USER: qcr
        image: rosbase
        container_name: rosbase
        stdin_open: true
        tty: true
        privileged: true
        environment:
            - DISPLAY
            - QT_X11_NO_MITSHM=1
            - ROS_MASTER=false
            - ROS_MASTER_URI=http://localhost:11311
        network_mode: host
        ipc: host
        user: qcr
        working_dir: /home/qcr
        volumes:
            - /tmp/.X11-unix:/tmp/.X11-unix:rw
            - /etc/timezone:/etc/timezone:ro
            - /etc/localtime:/etc/localtime:ro
        command: bash
``` 
## Dangling Docker Images and Forgetful Containers

A common _mess_ of docker-based work is the emergence of more and more containers that are not used. Many of these are due to applications that create them ignoring to have them destroyed (for example, visual studio code) and users who are forgetful.

To list the containers in the docker enginer (essentially the same):
```
docker ps

docker container ls
```
To remove a particular container:
```
docker rm <CONTAINER_NAME>
```
To remove all stopped containers:
```
docker container prune
```
Note: removing a container is irreverible - the storage is destroyed immediately (except the mounted volumes which come from somewhere).

Sometimes, the number of images can also get out of hand - again the untidyness of the user is to be blamed. 

To remove dangling images
```
docker rmi $(docker images -f "dangling=true" -q)
```

## Creating New Images for a Project

Refer to the [Setup of New Image](./docs/SETUP.md) for suggestions and tips.

## Running a Docker Image at System Bootup

At the QCR, some computer platforms are always the hosts of a particular docker image/container. In particular, end-users with little of no docker experience will appreciate a transparent docker setup.   

The following solution offers a way to get an image/container to be started without the user performing any action.

[Stack Overflow Solution](https://stackoverflow.com/questions/30449313/how-do-i-make-a-docker-container-start-automatically-on-system-boot/39493500#39493500)

## Hosting a Local Docker Hub

A local docker hub (docker image distribution) will be handy for computer platform management at the QCR. RAS engineers and QCR users can pull generic images and push project/user specific images.

Docker offers an official docker image for setting up such a local docker hub (called docker registry). Refer to teh following webpage for the instructions.

[How to Use Your Own Registry](https://www.docker.com/blog/how-to-use-your-own-registry-2/)

## Containarized Development

Refer to the [Workshop: Containerized Development with Docker and Visual Studio Code](./docs/CONTAINER_DEV.md) page.

## Developers

Dr Andrew Lui, Senior Research Engineer <br />
Dr Dasun Gunasinghe, Senior Research Engineer <br />
Robotics and Autonomous Systems, Research Engineering Facility <br />
Research Infrastructure <br />
Queensland University of Technology <br />
