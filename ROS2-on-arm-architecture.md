## Setup the development environment
The normal setup described in the following link is required for the cross-compilation -> https://github.com/ros2/ros2/wiki/Linux-Development-Setup

## Get the source
Create a workspace and clone all repos:

```
mkdir -p ros2_ws/src
cd ros2_ws
wget https://raw.githubusercontent.com/ros2/ros2/release-latest/ros2.repos
wget https://raw.githubusercontent.com/ros2-for-arm/ros2/master/ros2-for-arm.repos
wget https://raw.githubusercontent.com/ros2-for-arm/ros2/master/aarch64_toolchainfile.cmake
vcs-import src < ros2.repos 
vcs-import src < ros2-for-arm.repos
```

## Get an aarch64 toolchain and export it

```
sudo apt install g++-aarch64-linux-gnu gcc-aarch64-linux-gnu
export CROSS_COMPILE=aarch64-linux-gnu-
```

You can also pick a toolchain of your chosing. All you need to do is to set the CROSS_COMPILE environment variable accordingly.

## Remove Python support
No target filesystem with Python libraries is provided here, so Python is not supported (only C++). But if you have a copy of your target rootfs, you can set the CMAKE_SYSROOT variable to it and you would be able to use Python. 
A few packages must be modified to allow the compilation of ROS2:
- remove Python generator from `rosidl_default_generators`
	- `src/ros2/rosidl_typesupport/rosidl_default_generators/CMakeLists.txt`: comment `ament_export_dependencies(rosidl_generator_py)` (line 11) with a `#`
	- `src/ros2/rosidl_typesupport/rosidl_default_generators/package.xml`: comment `<buildtool_export_depend>rosidl_generator_py</buildtool_export_depend>`(line 16) with `<!-- ... -->`
- ignore examples/rclpy
	- `src/ros2/examples/rclpy` : add an empty file called `AMENT_IGNORE`
- ignore `test_msgs` (this one use the path to the python libraries during the install step and in this case try to install under `/test_msgs` because of the missing path !)
	- `src/ros2/rcl_interfaces/test_msgs` : add an empty file called `AMENT_IGNORE`
- ignore others repos (see list below)

You can do all of this with:

```
sed -e '/py/ s/^#*/#/' -i src/ros2/rosidl_typesupport/rosidl_default_generators/CMakeLists.txt
sed -i -r 's/<build(.+?py.+?)/<\!\-\-build\1\-\->/' src/ros2/rosidl_typesupport/rosidl_default_generators/package.xml
touch \
  src/ros/resource_retriever/AMENT_IGNORE \
  src/ros2/demos/AMENT_IGNORE \
  src/ros2/examples/rclpy/AMENT_IGNORE \
  src/ros2/geometry2/AMENT_IGNORE \
  src/ros2/kdl_parser/AMENT_IGNORE \
  src/ros2/orocos_kinematics_dynamics/AMENT_IGNORE \
  src/ros2/rclpy/AMENT_IGNORE \
  src/ros2/rcl_interfaces/test_msgs/AMENT_IGNORE \
  src/ros2/robot_state_publisher/AMENT_IGNORE \
  src/ros2/ros1_bridge/AMENT_IGNORE \
  src/ros2/rosidl/rosidl_generator_py/AMENT_IGNORE \
  src/ros2/rviz/AMENT_IGNORE \
  src/ros2/system_tests/AMENT_IGNORE \
  src/ros2/urdf/AMENT_IGNORE \
  src/ros2/urdfdom/AMENT_IGNORE \
  src/ros2/urdfdom_headers/AMENT_IGNORE
```

## Trigger a build
```
src/ament/ament_tools/scripts/ament.py build --force-cmake-configure --cmake-args -DCMAKE_TOOLCHAIN_FILE=`pwd`/aarch64_toolchainfile.cmake -DTHIRDPARTY=ON --
```

## Installation
Once the compilation is done you will have to move the generated libraries (`install/lib`) in your target filesystem.
The default path is under `/lib` but you can also choose another directory and set the `LD_LIBRARY_PATH` variable to it.