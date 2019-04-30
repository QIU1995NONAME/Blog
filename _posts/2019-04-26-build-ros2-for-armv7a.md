---
layout: docs
title:  "Build ROS2 for armv7a"
date:   1970-01-01
categories: Linux ROS2
enable_mathjax: false
enable_mermaid: false
enable_comments: false
---
system needed: tinyxml tinyxml2 log4cxx
``` bash
mkvirtualenv venv_ros2
workon venv_ros2
pip3 install -U vcstool colcon-common-extensions
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
wget https://raw.githubusercontent.com/ros2/ros2/release-latest/ros2.repos
wget https://raw.githubusercontent.com/ros2-for-arm/ros2/master/ros2-for-arm.repos

vcs-import src < ros2.repos
vcs-import src < ros2-for-arm.repos
rm ros2.repos ros2-for-arm.repos

cat > armv7a_toolchain.cmake << __EOF__
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_VERSION 1)
SET(CMAKE_SYSTEM_PROCESSOR armv7a)

SET(CMAKE_C_COMPILER   arm-unknown-linux-gnueabi-gcc)
SET(CMAKE_CXX_COMPILER arm-unknown-linux-gnueabi-g++)

SET(CMAKE_FIND_ROOT_PATH ${CMAKE_CURRENT_LIST_DIR}/install)
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
src/ros2-for-arm/example
src/ros-visualization
__EOF__

cat << __EOF__ | while read line; do touch ${line}/COLCON_IGNORE; done
src/ros-perception/laser_geometry
src/ros/resource_retriever
src/ros2/geometry2
src/ros2/urdf
src/ros2/kdl_parser
src/ros2/rmw_connext
src/ros2/orocos_kinematics_dynamics
src/ros2/robot_state_publisher
src/ros2/rviz
src/ros2/rcl/rcl/test
src/ros2/urdfdom
src/ros2/rclpy
src/ros2/rosidl_typesupport_opensplice
src/ros2/system_tests
src/ros2/rosidl_python
src/ros2/rmw_opensplice
src/ros2/rosidl_typesupport_connext
src/ros2/rcl_interfaces/test_msgs
src/ros/pluginlib/pluginlib
__EOF__



function __needed_DIR(){
    while [[ "x$1" != "x" ]]; do
        echo "-D${1}_DIR=`pwd`/install/share/${1}/cmake"
        shift
    done
}

colcon build                       \
    --continue-on-error            \
    --symlink-install              \
    --merge-install                \
    --cmake-force-configure        \
    --cmake-args                   \
    --no-warn-unused-cli           \
    -DTINYXML2_LIBRARY=/usr/arm-unknown-linux-gnueabi/usr/lib/libtinyxml2.so \
    -DTINYXML2_INCLUDE_DIR=/usr/arm-unknown-linux-gnueabi/usr/include        \
    -DLog4cxx_LIBRARY=/usr/arm-unknown-linux-gnueabi/usr/lib/liblog4cxx.so   \
    -DLog4cxx_INCLUDE_DIR=/usr/arm-unknown-linux-gnueabi/usr/include         \
    -DCMAKE_TOOLCHAIN_FILE=`pwd`/armv7a_toolchain.cmake \
    -DTHIRDPARTY=ON -DBUILD_TESTING:BOOL=OFF            \
    $(__needed_DIR poco_vendor console_bridge_vendor libyaml_vendor) \
    $(__needed_DIR ament_cmake_core ament_cmake_export_dependencies ament_cmake_test) \
    $(__needed_DIR ament_cmake_libraries ament_cmake_export_definitions) \
    $(__needed_DIR ament_cmake_export_interfaces ament_cmake_export_libraries) \
    $(__needed_DIR ament_cmake_export_link_flags ament_cmake_include_directories) \
    $(__needed_DIR ament_cmake_target_dependencies ament_cmake_python) \
    $(__needed_DIR ament_cmake_export_include_directories ament_cmake) \
    $(__needed_DIR ament_cmake_ros ) \
    $(__needed_DIR rosidl_typesupport_interface rosidl_typesupport_introspection_c) \
    $(__needed_DIR rosidl_generator_c rosidl_generator_cpp rosidl_cmake rosidl_adapter) \
    $(__needed_DIR rcutils fastcdr fastrtps_cmake_module fastrtps) \
    $(__needed_DIR rmw rosidl_typesupport_fastrtps_cpp rmw_implementation_cmake) \
    $(__needed_DIR rmw_fastrtps_shared_cpp rosidl_typesupport_fastrtps_c) \
    $(__needed_DIR rosidl_typesupport_introspection_cpp rmw_fastrtps_cpp) \
    $(__needed_DIR rosidl_default_generators rosidl_default_runtime) \
    $(__needed_DIR builtin_interfaces unique_identifier_msgs rcl_interfaces) \
    $(__needed_DIR std_msgs rmw_implementation action_msgs rosgraph_msgs) \
    $(__needed_DIR geometry_msgs stereo_msgs map_msgs nav_msgs) \
    $(__needed_DIR rosidl_actions rcl_logging_noop sensor_msgs) \
    $(__needed_DIR rcl lifecycle_msgs map_msgs rcl_yaml_param_parser) \
    $(__needed_DIR rosidl_typesupport_c rosidl_typesupport_cpp rclcpp) \
    $(__needed_DIR rcl_action rcl_lifecycle python_cmake_module tlsf) \
    $(__needed_DIR ) \
    $(__needed_DIR ) \
    -Dyaml_DIR=`pwd`/install/cmake \
    -DFastRTPS_INCLUDE_DIR=`pwd`/install/include \
    -DFastRTPS_LIBRARY_RELEASE=`pwd`/install/lib/libfastrtps.so \
    -Dconsole_bridge_DIR=`pwd`/install/lib/console_bridge/cmake \
    -DPoco_DIR=`pwd`/install/lib/cmake/Poco \
    -DPocoFoundation_DIR=`pwd`/install/lib/cmake/Poco


    -DCMAKE_BUILD_RPATH="`pwd`/build/poco_vendor/poco_external_project_install/lib/;`pwd`/build/libyaml_vendor/libyaml_install/lib/"


find -type d | grep site-packages | grep -v lib64 \
| while read line ; do
    __i=$(dirname $(readlink -fn $line/..))
    rm -rfv $__i
    ln -svf lib64 $__i
done

```


CBUILD=x86_64-pc-linux-gnu
CHOST=
HOSTCC=${CBUILD}-gcc
ROOT=

ACCEPT_KEYWORDS="${ARCH}"

USE="${ARCH} -pam -fortran"

CFLAGS="-O2 -pipe -fomit-frame-pointer"
CXXFLAGS="${CFLAGS}"

FEATURES="-collision-protect sandbox buildpkg noman noinfo nodoc"

GENTOO_MIRRORS="http://mirrors.163.com/gentoo/"
DISTDIR="/usr/portage/distfiles/"
PKGDIR="/usr/portage/packages/${CHOST}/"
PORTAGE_TMPDIR="/var/tmp/${CHOST}/"
"

