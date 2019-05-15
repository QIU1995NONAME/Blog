---
layout: docs
title:  "Build ROS2 for armv7a"
date:   2019-04-26
categories: Linux ROS2
enable_mathjax: false
enable_mermaid: false
enable_comments: false
---
system needed:  tinyxml2 log4cxx
``` bash
mkvirtualenv venv_ros2
workon venv_ros2
pip3 install -U vcstool colcon-common-extensions
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
wget https://raw.githubusercontent.com/ros2/ros2/release-latest/ros2.repos
vcs-import src < ros2.repos
rm ros2.repos

cat > armv7a_toolchain.cmake << __EOF__
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_VERSION 1)
SET(CMAKE_SYSTEM_PROCESSOR armv7a)

SET(CMAKE_C_COMPILER   arm-unknown-linux-gnueabi-gcc)
SET(CMAKE_CXX_COMPILER arm-unknown-linux-gnueabi-g++)

SET(CMAKE_FIND_ROOT_PATH /usr/arm-unknown-linux-gnueabi)
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
SET(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

SET(THREADS_PTHREAD_ARG "0" CACHE STRING "Result from TRY_RUN" FORCE)

__EOF__


cat << __EOF__ | while read line; do touch ${line}/COLCON_IGNORE; done
src/ros2/ros1_bridge
src/ros2/demos
src/ros2/examples
src/ros-visualization
src/ros/urdfdom_headers
src/ros2/rcl/rcl/test
src/ros2/rviz
src/ros2/rcl_interfaces/test_msgs
src/ros2/system_tests
src/ros2/orocos_kinematics_dynamics
src/ros2/kdl_parser
src/ros-perception/laser_geometry
src/ros2/rosidl_typesupport_connext
src/ros2/rosidl_typesupport_opensplice
src/ros2/rmw_connext
src/ros2/rmw_opensplice
src/ros2/urdfdom
src/ros2/urdf
src/ros2/geometry2
src/ros2/robot_state_publisher
__EOF__

vim ./src/ros2/realtime_support/tlsf_cpp/CMakeLists.txt
 ### 把 tlsf_allocator_example 相关的全部删掉

colcon build                                   \
    --continue-on-error                        \
    --merge-install                            \
    --cmake-force-configure                    \
    --cmake-args                               \
    --no-warn-unused-cli                       \
    -DCMAKE_BUILD_TYPE=Release                 \
    -DCMAKE_TOOLCHAIN_FILE=`pwd`/armv7a_toolchain.cmake \
    -DTINYXML2_LIBRARY=/usr/arm-unknown-linux-gnueabi/usr/lib/libtinyxml2.so \
    -DTINYXML2_INCLUDE_DIR=/usr/arm-unknown-linux-gnueabi/usr/include        \
    -DLog4cxx_LIBRARY=/usr/arm-unknown-linux-gnueabi/usr/lib/liblog4cxx.so   \
    -DLog4cxx_INCLUDE_DIR=/usr/arm-unknown-linux-gnueabi/usr/include         \
    -DTHIRDPARTY=ON -DBUILD_TESTING:BOOL=OFF            \
    $(find ./install/share -type d -name cmake | cut -d'/' -f4 | while read line;do
        echo "-D${line}_DIR=`pwd`/install/share/${line}/cmake"
    done) \
    -Dyaml_DIR=`pwd`/install/cmake \
    -DFastRTPS_INCLUDE_DIR=`pwd`/install/include \
    -DFastRTPS_LIBRARY_RELEASE=`pwd`/install/lib/libfastrtps.so \
    -Dconsole_bridge_DIR=`pwd`/install/lib/console_bridge/cmake \
    -DPoco_DIR=`pwd`/install/lib/cmake/Poco \
    -DPocoFoundation_DIR=`pwd`/install/lib/cmake/Poco

 ### 如果中间有问题出现，可以尝试下列命令
cd install
rm -rf lib64
ln -svf lib lib64

```


