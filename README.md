This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

### Team

This is the  team repo for the capstone project of Udacity's Self-Driving Car Engineer Nanodegree. The team members are:
* Stojan Gancev(gancevstojan@yahoo.com)
* Yi Yang (satine.yy@gmail.com)
* Gokberk Ozden (ozdengokberkk@gmail.com)

### Result check:
- Car smoothly follows waypoints in the simulator. 

- Respects the target top speed set for the waypoints' twist.twist.linear.x in waypoint_loader.py. Works by testing with different values (from 20 to 60) for kph velocity parameter in /ros/src/waypoint_loader/launch/waypoint_loader.launch.

- Stops at traffic lights when needed. However, due to the camera issue with the workspace and the time limit, the traffic lights information are read directly from  '/vehicle/traffic_lights', instead of the classfier.

- Stops and restarts PID controllers depending on the state of /vehicle/dbw_enabled.

- Publishes throttle, steering, and brake commands at 50hz.

- Launches correctly using the launch files provided in the capstone repo.

### Waypoint Updater Node

Waypoint Updater Node purpose is to publish a fixed number of waypoints ahead of the vehicle with the correct target velocities, depending on traffic lights. 
Waypoint Updater Node is subscribed to following topics:
- /base_waypoints : list of waypoints for the track.
- /current_pose : car current position.
- /traffic_waypoint : list of all detected trafic lights.

and publishing waypoints to topic :
- /final_waypoints : list of waypoints car should follow with the desired speed of the car in each waypoint. Number of waypoints we publish is defined in LOOKAHEAD_WPS currently set to 200. When traffic light is detected with stop signal we start decelerating car continuosly through the waypoints.

During testing with the simulator on the workspace enviroment we expirienced performance issues when update rate for /final_waypoints were set to 50 Hz, car reacted with huge latency not following the waypoints, and steers at wrong time. Since the waypoint follower is at 30Hz, so the rate here can be set to 30Hz or even less when waypoint updater ; currently set to 25Hz.
        
### DBW(Drive-by-wire) node

The DBW node takes the /twist_cmd topic as input and uses various controllers to provide appropriate throttle, brake, and steering commands. These commands can then be published to the following topics:
- /vehicle/steering_cmd
- /vehicle/throttle_cmd
- /vehicle/brake_cmd

They are published at 50Hz as required. Since a safety driver may take control of the car during testing, to avoid the PID controller mistakenly accumulating error, the DBW status is checked by by subscribing to /vehicle/dbw_enabled.

The commands above are calculated respectivelly:
#### steering:
The steering is calculated by yaw_controller, which takes the desired linear x velocity and angular z velocity of the twist command from waypoint follower, and the current linear x velocity as input:
- steering_angle = arctan(wheel_base / turning_radius) * steer_ratio
- turning_radius = current_linear_velocity / target_angular_velocity
- target_angular_velocity = desired_angular_velocity * (current_linear_velocity / target_linear_velocity)

The current linear velocity is filtered by a low pass-by filter to avoid noises.

#### throttle:
A PID controller is set for the throttle control. It takes in the difference between the desired linear velocity and the current linear velocity, as well as the sample time, and outputs the throttle(0. to 1.0).

#### brake:
While the acceleration depends on the throttle, the deceleration of the car relys on the brake. When the car needs to slow down, which means the desired velocity is less than the current velocity:
- throttle is set to 0
- brake = a_deceleration * vehicle_mass * wheel_radios

To fix the car wanding issue as mentioned in the walkthrough, the following_flag check is desactivated in waypoint_follower/src/pure_pursuit_core.cpp PurePursuit::calcTwist(), so that the waypoint follower updates the waypoint angular velocity all the time.

### Traffic Light Detection Node

Important Note: Obstacle Detection is not implemented.

The node has following input signals:
	1. current_pose : Vehicle position information
 2. base_waypoints : Waypoint the vehicle should follow
 3. vehicle/trafficlights: Coordinates of traffic lights
 4. image_color: Image stream from vehicle's camera to determine color of the traffic light ahead
    
The node returns two outputs:
	1. Closest waypoint for stopping at upcoming detected light
 2. ID of traffic light color

The nodes gets vehicle position and light positions, iterates through them to find the closest stop line ahead of the vehicle.
Once, the closest light is found node gets light state information. (For simulation state information is directly taken from TLDetector class)

Please use **one** of the two installation options, either native **or** docker installation.
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
To set up port forwarding, please refer to the "uWebSocketIO Starter Guide" found in the classroom (see Extended Kalman Filter Project lesson).

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

### Other library/driver information
Outside of `requirements.txt`, here is information on other driver/library versions used in the simulator and Carla:

Specific to these libraries, the simulator grader and Carla use the following:

|        | Simulator | Carla  |
| :-----------: |:-------------:| :-----:|
| Nvidia driver | 384.130 | 384.130 |
| CUDA | 8.0.61 | 8.0.61 |
| cuDNN | 6.0.21 | 6.0.21 |
| TensorRT | N/A | N/A |
| OpenCV | 3.2.0-dev | 2.4.8 |
| OpenMP | N/A | N/A |

We are working on a fix to line up the OpenCV versions between the two.
