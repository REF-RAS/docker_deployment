# Copyright 2024 - Andrew Kwok Fai LUI, Robotics and Autonomous System Group,
# Research Engineering Facility, RI
# the Queensland University of Technology

ARG ROS_DISTRO=noetic

FROM rosbase
LABEL author "Andrew Lui <luia2@qut.edu.au>"

ARG USER=qcr
ENV ROBOT_WS=/home/${USER}/robot_ws
WORKDIR /home/${USER}
SHELL ["/bin/bash", "-c"]

# Install armer workspace
RUN mkdir -p ${ROBOT_WS}/src
RUN cd ${ROBOT_WS}/src && \ 
git clone --branch master https://github.com/qcr/armer.git && \
git clone --branch main https://github.com/qcr/armer_msgs.git && \
git clone --branch main https://github.com/qcr/pointcloud_utils.git

# Add ros dependency
RUN cd ${ROBOT_WS} && rosdep install --from-paths src --ignore-src -r -y --rosdistro ${ROS_DISTRO} 

# Workspace create and make 
RUN source /opt/ros/${ROS_DISTRO}/setup.bash && \
cd ${ROBOT_WS} && \
catkin_make -DPYTHON_EXECUTABLE=/usr/bin/python3

# install python modules
RUN pip install -r ${ROBOT_WS}/src/armer/requirements.txt
# RUN cd ${ROBOT_WS}/src/armer && python3 setup.py build_ext --build-lib=armer/cython/
RUN pip install --force-reinstall spatialmath-python==1.0.0
RUN pip install --force-reinstal qpsolvers==3.4.0

# copy entry point script
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc
RUN echo "source ${ROBOT_WS}/devel/setup.bash" >> ~/.bashrc 

COPY assets/entrypoint_setup.sh /usr/local/bin/ros_catkin_entrypoint
RUN sudo chmod +x /usr/local/bin/ros_catkin_entrypoint
ENTRYPOINT ["/usr/local/bin/ros_catkin_entrypoint"]
CMD ["bash"]
