# ROS SLAM bot with Arduino, Rasbperry Pi
##### This project is for building a robot and programming it to autonomously navigate a pre-built map. 
##### I'll walkthrough setting up an autonomous ROS stack on a Raspberry Pi for Teleoperation, mapping, localization and navigation.

<img 
src="https://user-images.githubusercontent.com/49666154/145496403-223023c8-823d-47cd-9dff-706a4a9c5c3b.jpg" width="700px"  > 

### What is SLAM?

- #### One of the popular applications of ROS is SLAM(Simultaneous Localization and Mapping). The objective of the SLAM in mobile robotics is constructing and updating the map of an unexplored environment with help of the available sensors attached to the robot which is will be used for exploring.

### Lidar sensor and SLAM
<img src="https://user-images.githubusercontent.com/49666154/145659318-f82e87b2-7955-4e3e-bb09-61fa0aca6bfd.jpg" width="340px" > <img src="https://user-images.githubusercontent.com/49666154/145659281-7670655e-9465-4756-b84b-16ba846a1927.jpg" width="310px" > 

 #### lidar is a 360-degree two-dimensional laser range scanner (LIDAR). its commonly used to make high-resolution maps, with applications in surveying and many others. Its provided by different manufactures, Each one has its own configurations with ROS. In my case I used YDLidar X2L.
 

## Building the robot
### Required components:
###### 1-Rasbperry pi 4
###### 2-Arduino UNO
###### 3-YDLidar sensor (X2L)
###### 3-L298N H-bridge
###### 4-Four DC motors 3-6 volts with wheels
###### 5-10k mAh Power supply 
###### 6-9V battery

## Setting up Rasbperry Pi for ROS remote connection
##### Firstly the Rasbperry Pi needs a system that has ROS in order to work with YDLidar, To shortcut this step i've dowloaded the Ubuntu 16.04 Xenial with pre-installed ROS from Ubiquity Robotics. The instructions are explained on the website. Visit https://downloads.ubiquityrobotics.com/pi.html
##### My laptop machine is ubuntu 20.04 machine with ROS Noetic. Because this will be my main machine for ROS operations.
##### On Both machines, I've set the ROS_IP and ROSMASTER_URI in the 'bashrc', Its important in order to access ROS communication masseges.
- ##### On The Rasbperry Pi machine ROS_IP and ROS_MASTER_URI are same as the machine's network ip
- ##### On my laptop ROS_IP is the machine IP and ROS_MASTER_URI is the Rasbperry pi's IP. Because The ROS master node is going to run on the Rasbperry pi. 
##### example:
##### 
```` 
Rasbperry pi bashrc: export ROS_IP=192.168.200.228 export ROS_MASTER_URI=http://192.168.200.228:11311
Laptop bashrc: export ROS_IP=192.168.200.123 export ROS_MASTER_URI=http://192.168.200.228:11311
````
- ##### Connecting the Rasbperry Pi to the wifi network (same as laptop network )
```` 
$ pifi add YOURNETWOKNAME YOURNETWORKPASSWORD 
````
- ##### To connect to the Rasbperry Pi from my laptop over ssh: 
````
$ ssh ubuntu@'Rasbperry Pi IP'
````

## Testing Lidar with Rasbperry Pi:
again iam using the YDLIDAR X2L for this build. The first step is to install the necessary drivers which simply is a ROS package.
````
$ cd catkin_ws/src 
$ git clone https://github.com/YDLIDAR/ydlidar_ros
$ catkin_make
$ roscd ydlidar_ros/startup
$ sudo chmod 777 ./*
$ sudo sh initenv.sh
$ source catkin_ws/devel/setup.bash.
Run catkin_make again.
````
##### Then I tested the lidar with ``roslaunch ydlidar_ros lidar.launch``. and Visualize the result scans in my laptop machine with Rviz, by adding the topic /scan.

#
### L298N H-bridge motor driver  
<img src="https://user-images.githubusercontent.com/49666154/128776326-36a2416f-9356-49f9-842e-ab9bff2704f0.jpeg" width="300px" > <img src="https://user-images.githubusercontent.com/49666154/128803887-7bc041e8-9c74-42aa-8f75-aa2c68efa30d.png" width="400px" >
##### L298N is a dual H-Bridge motor driver which allows speed and direction control of two or four DC motors at the same time.
#
## Connection :
#### from arduino to L298N Driver
- ##### ~11 connected to ENA
- #####  12 connected to INA1
- #####  13 connected to INA2
- #####   8 connected to INB3
- #####   7 connected to INB4
- #####  ~9 connected to ENB
- ##### GND connected to GND

#### Motors and Power supllies:
 ##### Two DC motors are conected to each side of L298N in parallel:
- ##### Dc motors right side: Positive to out4,Negative to out3
- ##### Dc motors left side: Positive to out1,Negative to out2
- #####  9V Battery connected to L298N GND and VCC
- #####  10k mAh power supply connected to Rasbperry pi4
- ##### Arduino and  YDLidar are powerd by the rasbperry pi usb ports.

## Circuit Diagram:
##### Arduino and L298N
![Screenshot (254)](https://user-images.githubusercontent.com/49666154/145664909-43aead89-b663-4c01-b140-507947246565.png)
##### The Arduino and YDLidar are connected to the rasbperry pi usb ports as shown
<img src="https://user-images.githubusercontent.com/49666154/145665077-49dca7bd-78b3-4274-acd1-1f0f2671fb93.jpg" width="500px" >

## ROS Operations:
- #### Launch rosserial socket node so that the ESP32 connects to the machine:
> roslaunch rosserial_server socket.launch
- #### Launch Keybord node to control the robot: 
> rosrun teleop_twist_keyboard teleop_twist_keyboard.py

 ###### Note: The program will switch Off the ESP32 blue led only when It's connected to the Rosserial server in the machine.

