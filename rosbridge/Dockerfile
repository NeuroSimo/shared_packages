FROM osrf/ros:iron-desktop@sha256:eddad5f79aa7e888af29793c11f0ee44280c656b74dba0d37b2f48a0aa57352e

SHELL ["/bin/bash", "-c"]

# Install dependencies

RUN apt-get update && \
    apt-get install -y \
    dos2unix \
    python3-pip \
    ros-iron-ament-cmake-mypy \
    ros-iron-rmw-cyclonedds-cpp && \
    rm -rf /var/lib/apt/lists/*

RUN pip install \
    pymongo \
    tornado

# Set the Cyclone DDS configuration
COPY ros2_ws/src/shared_packages/resources/cyclonedds.xml /config/
ENV CYCLONEDDS_URI=file:///config/cyclonedds.xml

# Copy ROS workspace and build packages

WORKDIR /app/ros2_ws

# Copy shared ROS interfaces
COPY ros2_ws/src/shared_packages/interfaces src/interfaces

# Copy project-specific interfaces
ARG INTERFACES_PATH
COPY ${INTERFACES_PATH} src/interfaces

RUN source /opt/ros/iron/setup.bash && colcon build

# Copy the rosbridge node
COPY ros2_ws/src/shared_packages/rosbridge/rosbridge_suite src/shared_packages/rosbridge/rosbridge_suite

# Change line endings for rosbridge files
RUN find src/shared_packages/rosbridge -type f -exec dos2unix {} +

# Build the rosbridge node after building interfaces
RUN source /opt/ros/iron/setup.bash && colcon build --packages-up-to rosbridge_server

# Copy ROS entrypoint
COPY ros2_ws/src/shared_packages/rosbridge/ros_entrypoint.sh .
RUN chmod +x ros_entrypoint.sh

# XXX: Hack around the symlink issue, allows fail with || : as this isn't needed on linux
RUN cp install/rosbridge_server/lib/rosbridge_server/rosbridge_websocket.py install/rosbridge_server/lib/rosbridge_server/rosbridge_websocket || :

# Define entrypoint

ENTRYPOINT ["./ros_entrypoint.sh"]
