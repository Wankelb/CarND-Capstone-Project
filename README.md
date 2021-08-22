 <p align="center">
<img src=".//bild22.jpeg">
</p> 



# CarND-Capstone-Project
# System Integration
The aim of this project is to implement a ROS-based core of an autonomous vehicle. The vehicle must be able to drive through a closed test track, recognize the traffic light and stop if necessary. The code is evaluated in a Unity simulator and a real Lincoln MKZ

This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car.

Please use **one** of the two installation options, either native **or** docker installation.

Specifications

The car should:

- Smoothly follow waypoints in the simulator.
- Respect the target top speed set for the waypoints' twist.twist.linear.x in waypoint_loader.py. 
- Stop at traffic lights when needed.
- Stop and restart PID controllers depending on the state of /vehicle/dbw_enabled.
- Publish throttle, steering, and brake commands at 50hz.

# Video of Simulation
<p align="center">
<img src=".//vid29.gif">
</p> 


# System(ROS) Architecture
The following is a system architecture diagram showing the ROS nodes and topics used in the project.
The three core components of any good robot are the following:

Perception: Sensing the environment to perceive obstacles, traffic hazards as well as traffic lights and road signs.

Planning: Route planning to a given goal state using data from localization, perception and environment maps.

Control: Actualising trajectories formed as part of planning, in order actuate the vehicle, through steering, throttle and brake commands.

 <p align="center">
<img src=".//ros_nodes.png">
</p> 

# ROS Nodes Description
Here are the descriptions of the ROS nodes.
 <p align="center">
<img src=".//node_tld.png">
</p> 

**Traffic Light Detection Node( tl_detector):** This package contains the traffic light detection node: tl_detector.py. This node saves data from topics /image_color, /current_pose and /base_waypoints and broadcasts red traffic light stops in topic /Traffic_waypoints. The topic /current_pose contains the current location of the vehicle and /base_waypoints contains the complete list of waypoints that the vehicle will follow. You create both a traffic light detection node and a traffic light classification node.

 <p align="center">
<img src=".//node_wpu.png">
</p> 

**Waypoint Updater node (waypoint_updater):** This node sends the next 200 waypoints that are closest to and ahead of the vehicle's current location. This node also takes into account obstacles and traffic lights to adjust the speed of each waypoint.

This node subscribes to the following topics:
- base_waypoints: Waypoints for the entire track are published on this topic. This publication is a one-time process. The waypoint update node receives these waypoints, stores them for later use, and uses these points to extract the next 200 points ahead of the vehicle.

- traffic_waypoint: To get the index of the waypoint in the base_waypoints list that is closest to the red light so that the vehicle can be stopped. The waypoint update node uses this index to calculate the distance from the vehicle to the traffic light when the traffic light is red and the car needs to be stopped.

- current_pose: To get the current position of the vehicle.
The function of the waypoint update node is to process the waypoints provided by the waypoint loader and to prepare the next waypoints for the vehicle. The speed will be adjusted if there is a red light in front of you.

For the described process, if the track waypoints are already loaded, the following steps are followed:

- Identify the position of the car in the car (pose_cb): If one knows the position of the car in (x, y) -coordinates, the closed-track point is returned as an index, which is ordered according to its Euclidean distance. Then the next few points in front of you (defined by the constant LOOKAHEAD_WPS) are the last unprocessed waypoints.

- Processing of waypoints (waypoints_process): The function runs through a loop through the following waypoints and the following options can run.

- Traffic light not close or green: The speed of the waypoints is updated with the maximum permissible speed

- Red and close traffic lights: the car has to stop. The speed is set to 0. set

- Red traffic light in delay distance: car is approaching the traffic light, but is not yet that close. The speed decreases linearly.

After the waypoints are updated, they are published and sent via the waypoint follower to the twist controller which implements the actuator commands. This node publishes on the following topics:

- final_waypoints: Selected more than 100 waypoints including their speed information are published in this topic.

<p align="center">
<img src=".//node_dbw.png">
</p> 

**Twist_controller:** Carla is equipped with a wired drive system (dbw), which means that gas, brakes and steering are electronically controlled. This package contains the files responsible for controlling the tool: the dbw_node.py node and the twist_controller.py file, as well as a PID and low pass filter that you can use in your application. dbw_node subscribes to the / current_velocity topic with the / twist_cmd topic to get the target linear and angular velocities. Also, this node subscribes to / tool / dbw_enabled, which indicates whether the vehicle is in dbw or in driver control. This node outputs gas, braking and steering commands to / veh / gas_cmd, / vehicle / brake_cmd and / vehicle / steer_cmd.

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images

