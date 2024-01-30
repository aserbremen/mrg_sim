# Multi-Robot Graph SLAM using LIDAR Simulation Package

This repository contains a world and launch files for testing the the [Multi-Robot Graph SLAM Framework](https://github.com/aserbremen/Multi-Robot-Graph-SLAM) in a simulated environment using Gazebo (tested on ROS2 humble and Gazebo Fortress).

## Dependencies

This package depends on the following packages:

- [Gazebo Fortress](https://gazebosim.org/docs/fortress/install)
- [ros_gz](https://github.com/gazebosim/ros_gz/tree/humble)

```
sudo apt install ros-humble-ros-gz
```

## Usage

Launch the world in gazebo with the following command. Note that the first call to gazebo might take a while, since it has to download the models of the fortress.

```
ros2 launch mrg_slam_sim rubicon.launch.py
```

Spawn two robots equipped with LIDAR sensors and bridge the sensor data to ROS2 topics. Also start the teleop_joystick node to control the first robot atlas with a joystick (xbox). This terminal has to be kept active.

```
ros2 launch mrg_slam_sim dual_robot_sim.launch.py
```

If you don't have a joystick, you can disable the teleop_joystick node in the [dual robot config](config/dual_robot_sim.yaml) by setting `enable_teleop_joy` to false. Then you can control the robot with the keyboard using the following command:

```
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r __node:=teleop_twist_keyboard_node_atlas -r /cmd_vel:=/atlas/cmd_vel
```

Second robot bestla can be controlled with the keyboard in a new terminal using the following command:

```
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r __node:=teleop_twist_keyboard_node_bestla -r /cmd_vel:=/bestla/cmd_vel
``` 

Now to test the Multi-Robot Graph SLAM Framework with the [mrg_slam](https://github.com/aserbremen/mrg_slam) package, run the following command:

```
ros2 launch mrg_slam mrg_slam.launch.py model_namespace:=atlas x:=-7.0 y:=0.0 z:=3.96 # terminal 1
ros2 launch mrg_slam mrg_slam.launch.py model_namespace:=bestla x:=0.0 y:=-15.0 z:=3.8 # terminal 2
```

Or use two instances of Multi-Robot Graph SLAM inside two docker containers. Make sure that the docker containers can communicate with the host machine. For example, environment variables like ROS_LOCALHOST_ONLY or ROS_DOMAIN_ID should not be set. Then run the following commands:

``` 
docker run -it --rm --network=host --ipc=host --pid=host -e MODEL_NAMESPACE=atlas -e USE_SIM_TIME=true -e X=-7.0 -e Y=0.0 -e Z=4 --name atlas_slam andi/mrg_slam:0 # terminal 1
docker run -it --rm --network=host --ipc=host --pid=host -e MODEL_NAMESPACE=bestla -e USE_SIM_TIME=true -e X=0.0 -e Y=-15.0 -e Z=3.8 --name bestla_slam andi/mrg_slam:0 # terminal 2
```

Move the robots and visualize the results in rviz2:

```
rviz2 -d /path/to/mrg_slam/rviz/mrg_slam.rviz --ros-args -p use_sime_time:=true
```
