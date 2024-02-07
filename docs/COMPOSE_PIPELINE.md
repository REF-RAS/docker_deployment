# Docker Robot Arm Compose Pipeline

## Base Image Build Steps

Prior to creating the required robot image, it is recommended you build the base images required, this may be `rosbase`, `ros2base`, `moveit`, or `moveit2`. To build, simply run the following command:
```bash
# Ensure you have cloned and entered the dockerise package
# NOTE: assumes the package was cloned to the user's home folder
cd ~/docker_deployment

# Replace <BASE_PACKAGE> with one of the options named above
# NOTE moveit depends on rosbase - so build rosbase first
# NOTE moveit2 depends on ros2base - so build ros2base first
docker compose build <BASE_PACKAGE>
```

## Environment File Update
Once the base packages have been built, copy the ***.env_template*** to ***.env*** using the below command. This will enable its use through docker:
```bash
cp .env_template .env
```
*NOTE: the .env file is ignored by git, which enables it to be customised as needed*

Edit the ***.env*** file with an editor of your choice. Update the available environment variables with specific values for your robot and project. The first three environment variables define the robot image to be created. You can see the format from the table above that concatenates \<ROBOT>, \<ROSVER>, and \<COMMANDER>. A simple breakdown of each variable is noted below:
- __ROBOT__: this is the robot type, which can be xarm, panda, or abb. Please refer to the above table for available options
- __ROSVER__: this is the ROS version (Noetic or Humble) for ROS1 or ROS2, respectively.
- __COMMANDER__: this is the commander type for the robot, either moveit or armer

The remaining environment variables are defining paths and package names to include in the running container for use in project specific scenarios. A breakdown of the meaning behind each is provided below:
- __WORKSPACE_PATH__: this is the path (on the host machine) to where your packages are located. By default it assumes all the packages are in one location.
- __SCENE_PKG__: this is the name of the scene package; a package that defines the robot (and scene objects) as standard descriptions for use in the application
- __CONFIG_PKG__: this is the commander configuration package, which is mostly for a moveit implementation
- __COMMANDER_PKG__: this is the commander package that interfaces and commands our robot commander type (moveit or armer).

### Use of Environment Variables in Docker Compose File
The described variables above are used in the main __docker-compose.yaml__ file. This presents an example of how to define and use them to reduce the code overhead when adding and creating new robot images. 

*NOTE: more environment variables can be defined and then used in the __docker-compose.yaml__ as required. The above are simple starting variables.*

## Testing Updated Environment Variables
Once you have updated the __.env__ file for your project purposes, you can test that docker compose will work using the following command:

```bash
docker compose config
```

This should output errors (if the variables were incorrectly inputted or if there is a structure issue between the .env and docker-compose.yaml files). One success, a print of all the available images (and their loaded compose components) will be provided.

## Building and Running a Robot Arm Image
Having now updated the required variables, and tested that it works, you can build the robot arm image and container. As can be observed in the __docker-compose.yaml__ file, the provided initial service is called *robot-service*, as the environment variables will populate based on user defined input. This keeps the command needed simple and reusable. To build simply run the following command:
```bash
docker compose build robot-service
```

Once built, to run, simply run the following command:
```bash
docker compose up robot-service
```

When running, you will notice that the \<ROBOT> variable is used to create the container name, which will be in the format *\<ROBOT>_container*. Therefore, to attach to your running container, simply run the following command to open a bash terminal:
```bash
docker exec -it <ROBOT>_container bash
```
