## Capstone Project - Programming a Real Self-Driving Car
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

In this project our team developed a integrated system using ROS, composed by the main components of CARLA - Udacity Self-Driving Car

## Project Team Members
|  Name                                   | Udacity Account Email Address     |
|:---------------------------------------:|:---------------------------------:|
| Jefferson Nascimento                    |   jnsagai@outlook.com             |
| Pratima Nagare                          |   nagarepratima27@gmail.com       |

[//]: # (Image References)

[image1]: ./images/final-project-ros-graph.png "Carla's System Architecture"
[image2]: ./images/test%20predictions/1.png "Test Prediction 1"
[image3]: ./images/test%20predictions/2.png "Test Prediction 2"
[image4]: ./images/test%20predictions/3.png "Test Prediction 3"
[image5]: ./images/graphs/loss.png "Loss"
[image6]: ./images/graphs/mAP.png "mAP"

## Architecture Overview

Carla's Software Architecture is based on [ROS](https://www.ros.org/) which is a flexible framework for writing robot software. It is a collection of tools, libraries, and conventions that aim to simplify the task of creating complex and robust robot behavior across a wide variety of robotic platforms.
Following the state-of-car architecture for self-driving cars, Carla's archicture can be divided in 3 main domains:
 - **Perception**: The process of perception in self-driving cars uses a combination of high-tech sensors and cameras, combined with state-of-the-art software to process and comprehend the environment around the vehicle, in real-time. In the case of Carla, Perception contains a component for obstacle detection and also a Traffic Light Detection.
 - **Planning**: Here is where the decisions are made. It’s about implementing the brain of an autonomous vehicle. Here the Waypoint Loader Software Component and Waypoint Updater are implemented.
 - **Control**: The Control step consists of following the trajectory generated as faithfully as possible. A path is a sequence of waypoints each containing a position (x; y) an angle (yaw) and a speed (v). The purpose of a controller is to generate instructions for the vehicle such as steering wheel angle or acceleration level taking into account the actual constraints and the trajectory generated. In Carla the Control domain is represented by a Drive-by-Wire Software component and a Waypoint Follower.

![alt text][image1]

## Perception
### Traffic Light Detection
 The detector tries to find the closest traffic light to the car, classify it's state(RED, YELLOW, GREEN) and in case of a RED light, it publishes it's waypoint to `/traffic_waypoint`. 
#### Dataset
The original data is raw without labels or annotations, we found this dataset where the images were annotated and converted to TFRecord, hence we used it to train the object detector The data can be found [here](https://github.com/vatsl/TrafficLight_Detection-TensorFlowAPI#get-the-dataset). We have trained only for simulator data but same can be done for real data.

#### Training
Model used : ssd_inception_v2_coco_2018_01_28 </br>
Tensorflow version: 1.15(GPU) </br>
Steps to train: https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf1.md </br>
Pipeline Config (Tensorflow Object Detection API) used: https://github.com/jnsagai/Traffic_Light_Detector/blob/main/traffic_light_classification.config </br>
Label Map Used: https://github.com/jnsagai/Traffic_Light_Detector/blob/main/label_map.pbtxt.txt

Some test image results:

![alt text][image2]

![alt text][image3]

![alt text][image4]

Training graphs:
![alt text][image5]

![alt text][image6]

## Planning
### Waypoint Loader
This package loads the static waypoint data and publishes to `/base_waypoints`.

### Waypoint Updater Node
This package contains the waypoint updater node: waypoint_updater.py. The purpose of this node is to update the target velocity property of each waypoint based on traffic light and obstacle detection data. This node will subscribe to the `/base_waypoints`, `/current_pose`, `/obstacle_waypoint`, and `/traffic_waypoint topics`, and publish a list of waypoints ahead of the car with target velocities to the /final_waypoints topic.

This node publishs waypoints from the car's current position to some `x` distance ahead.
The first step was to implement a version which does not care about traffic lights or obstacles. Afterwards, this node was updated in order to use the status of traffic lights too in order to react and stop whenever a Red light is detected.
Every 100 miliseconds (10Hz) the following tasks are performed:
1.  Find the nearest `n` waypoints ahead of the vehicle where `n` is defined as a set number of waypoints.
2.  Determine if a red traffic light falls in the range of waypoints ahead of the traffic light.
3.  Calculate target velocities for each waypoint.
4.  Publish the target waypoints with velocities to the `final_waypoints` topic.

## Control
### DBW Node ###
Once messages are being published to `/final_waypoints`, the vehicle's waypoint follower will publish twist commands to the `/twist_cmd topic`. The goal for this package is to implement the drive-by-wire node which will subscribe to `/twist_cmd` and use various controllers to provide appropriate throttle, brake, and steering commands. These commands are then being published to the following topics:
`/vehicle/throttle_cmd`
`/vehicle/brake_cmd`
`/vehicle/steering_cmd`

### Waypoint Follower ###
This package containing code from [Autoware](https://github.com/CPFL/Autoware) which subscribes to `/final_waypoints` and publishes target vehicle linear and angular velocities in the form of twist commands to the `/twist_cmd` topic.

## Result ##
You can see a video of the car driving [here](https://drive.google.com/file/d/12lf-XgZZydiGOqLZ5wWNdB-LzSx2UQxk/view?usp=sharing)

## Native Installation

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
