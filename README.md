# RobSenCom
This repository will include the necessary resources for RobSenCom project. You can create issue on any of your questions regarding the materials, installation problems.

# Simulation Setup
Prerequisities:
```
rosdep install --from-paths ros2-linux/share --ignore-src --rosdistro $CHOOSE_ROS_DISTRO -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 osrf_testing_tools_cpp poco_vendor rmw_connext_cpp rosidl_typesupport_connext_c rosidl_typesupport_connext_cpp rti-connext-dds-5.3.1 tinyxml_vendor tinyxml2_vendor urdfdom urdfdom_headers"
sudo apt install -y libpython3-dev
```
Clone the github repository to the /src folder of the ros2_ws and follow the listed commands below to setup the enviornment:
```
cd ~/ros2_ws
. ~/ros2_ws/install/local_setup.bash
rosdep install -i --from-paths . -y
colcon build
```
1. Install <a href="https://www.nsnam.org/wiki/Installation">NS-3</a>
2. Install <a href="http://classic.gazebosim.org/tutorials?tut=ros2_installing&cat=connect_ros">gazebo_ros_pkgs for ROS2</a> 
3. Modify the configuration file within /synchronization_module/config with the ips of the agents and choice of ROS2 QoS policy.
4. First, launch the network simulator by executing the following command:
```
ros2 run synchronization_module net 
```
5. The network module will be put on hold until the rest of the components are running. Next, in another terminal, launch the physics module by executing the following command:
```
ros2 run synchronization_module phy 
```
The physics module generates random positions for the agents and the network modules controls the packets and delays them for 300ms.

6. Again, this component will hold until the rest of the simulation is launched. Finally, launch the coordinator file to start the synchronization process by executing the command:
```
ros2 run network_module coordinator
```
We sincerely acknowledge the <a href="https://arxiv.org/abs/2101.10113">ROS-NetSim</a> paper for valuable direction to implement this work.

# Preliminary Experimentation Files
The /preliminary_tests_system repository contains two files, one named *sender.bash* and another named *receiver.bash* these two files were run on separate robotic agents to generate packet loss results in varying architectures. The only dependencies for the experiment are 
```
- tcpdump
- netcat
- a robot agent with ROS installed and a valid topic currently running. 
```
The topic to be subscribed to and duration for which the sender will record the topic information can be modified in the constants at the top of the *sender.bash* file, similarly IP addresses of robotic agents in experiment have to be specified in the provided files.
To run the experiment the following steps should be followed
```python
# first initiate receiver for listening
./receiver.bash
# next initiate sender to send a file containing topic data given
./sender.bash
# record results from opened terminals and rerun experiment to generate an average packet loss
```
Comparison of TCP and UDP is based on packet loss probability trends that vary with agent distance in a multi-agent environment. This system uses a master-based (ROS1) architecture generated using the code described above.
![Preliminary Results](preliminary_results-1.png)

# Package for Testing Performance of Ros2 system
The github repository https://github.com/irobot-ros/ros2-performance/tree/foxy contains a Ros2 performance evaluation package developed by irobot that provides a framework for performance testing of a ROS2 system, however the Ros2 topics used during testing are not useful ie. randomly generated data, the framework is a worthy endeavor for future modifications to evaluate network performance of a Ros2 architecture.
The following subsections describe steps used for installation of the package to be compatible on turtlebot3 burger robotic agents, the installation process uses Ros2 foxy as that is currently the most recent supported Ros2 version for turtlebt3 burger robot agents

## Installing GUI for the turtlebot3

After running the following commands the turtlebot3 should now be equipped with a GUI that makes ease of use much easier
```
sudo apt-get install ubuntu-desktop-minimal
sudo reboot
```
Since the network manager before was netplan, in order to use NetworkManager the netplan config file found in */etc/netplan/50-cloud-init.yaml* has to be modified using the following steps in terminal:
````python
sudo nano /etc/netplan/50-cloud-init.yaml
# Convert the file to look like this
```
network:
            version: 2
            renderer: NetworkManager
```
# CTRL+S, CTRL+X to save and quit
sudo netplan apply
sudo reboot
````
Now the network can be managed using the drop down menu at the top right of the screen

## Installing and running performance package on two Turtlebot3 Burger robots
When running a system using two separate turtlebots in a ROS2 foxy system there are two changes that must be added before the performance package can be successfully installed. 

### Adding a Namespace to the turtlebots
The easiest way to separate the nodes and topics of two turtlebots is the use of namespaces. This is done by changing a total of 3 files in the turtlebot3 and LDS packages provided by robotis, starting with the node declaration found in *turtlebot3/turtlebot3_bringup/launch/robot.launch.py* (*tb3_XXX* represents any namespace that should be added to the node and topics associated with it).
```python
Node(
            package='turtlebot3_node',
            namespace='tb3_XXX',          // add this line
            executable='turtlebot3_ros',
            parameters=[tb3_param_dir],
            arguments=['-i', usb_port],
            output='screen'),
```
To complete of adding the namespace the param file must be changed in *turtlebot3/turtlebot3_bringup/param/burger.yaml'
```python
tb3_XXX:                              // add this line at top of file, properly indent following lines
            turtlebot3_node:
                        ...
            
```
Similarly a namespace must be added to the provided lidar package to separate the scan topics completed by changing the node declaration in launch file *ld08_driver/launch/ld08.launch.py*
```python
Node(
            package='ld08_driver',
            namespace='tb3_XXX',    //add this line
            executable='ld08_driver',
            name='ld08_driver',
            output='screen'),
```
After building the package and running the bringup, nodes will now be separated by a namespace ex: *tb3_XXX/scan, tb3_XXX/cmd_vel, tb3_XXX/...*

### Adding virtual memory before running colcon build
Since the raspberry pi's found on the turtlebots don't have a high amount of RAM it's important to add virtual memory before building the performance package or the build process will freeze. This can be done by installing the dphys-swapfile package using the terminal.
```
sudo apt install dphys-swapfile
sudo dphys-swapfile swapoff
sudo dphys-swafile swapon
```
Turning the package off and on usually will suffice however it may require a system reboot before going into affect, the virtual memory can be viewed by running *free -m* which will now display swap memory of around 2.4 GB. To increase or reduce the memory the following steps should be taken:
````
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile
```
Change the CONF_SWAPSIZE value
CTRL + S, CTRL + X to save and exit
```
sudo dpys-swapfile swapon
````
### Installing the Performance Package
After successfully completing the steps to setup the two turtlebots, the performance package can be successfully built and installed. This may take several minutes to complete however adding the virtual memory will prevent any freezing.

# Hardware
- Duckiedrone
https://docs.duckietown.org/daffy/opmanual_sky/out/index.html
- Turtlebot3 Burger
https://www.robotis.us/turtlebot-3-burger-us/
