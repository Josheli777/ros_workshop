# ROS Workshop - Tutorial 6 - Turtlebot3 Navigation
## ME4140 - Introduction to Robotics, ME6640 - Advanced Robotics 

## Navigation:

What do we mean by navigation? This means different things in different places. Here, we mean the navigation stack in ROS melodic. This tutorial comes from [here](http://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#simulation) 

<img src="turtlebot3_models.png" alt="drawing" width="400"/>

(Image: [emanual.robotis.com](https://emanual.robotis.com/docs/en/platform/turtlebot3/features/#features) )

	
## Overview:
After completing _Tutorial 5 - Turtlebot Simulator_, You have learned some basics of ROS, and you have a more advanced robot. Next you are going to learn to use the navigation stack with the turtlebot3 simulator. Read more [here](https://emanual.robotis.com/docs/en/platform/turtlebot3/navigation/#ros-1-navigation) and [here](http://wiki.ros.org/navigation/Tutorials).
	
## System Requirements:

- **ROS+OS**: This tutorial is intended for a system with ROS Melodic installed on the Ubuntu 18.04 LTS operating system. Alternate versions of ROS (i.e. - Kinetic, Noetic, etc.) may work but have not been tested. Versions of ROS are tied to versions of Ubuntu.
- **ROS**: Your computer must be connected to the internet to proceed. Update the system before you begin.
- **Workspace Setup**: The Turtlebot3 Simulator from tutorial 5 must be operational before completing tutorial 6.  

	
## Before Beginning:
	
- **Backup the System:** If you are using a virtual machine, it is recommend to make a snaphot of your virtual machine before you start each module. In the event of an untraceable error, you can restore to a previous snapshot. 

- **ROSLAUNCH:** This tutorial involves using the roslaunch command which runs a muliple of nodes at once as described in the launch file. We will learn more about this later. 

- **Mouse for 3D viewing:** This simulator view is much easier use if you have a three button mouse plugged in, but this is not required.
	
		
## Part 1 - Install navigation and gmapping packges
	

### Step 1 Install `navigation` and `gmapping` 
Install the packages required for autonomous navigation if you have not already. It will not hurt to run the install command again.

```
sudo apt update

sudo apt install ros-melodic-navigation ros-melodic-gmapping
```

### Step 2 -  Set the robot model. This only needs to be done once. Modify the .bashrc file If you want to change models.

```
echo "export TURTLEBOT3_MODEL=waffle_pi" >> ~/.bashrc
source ~/.bashrc
```

## Part 2 - Generate a map of the virtual space:



### Step 1 Start the turtlebot3 simulator

```
roslaunch turtlebot3_gazebo turtlebot3_world.launch
```

### Step 2 Start gmapping SLAM
 
```
roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping
```

### Step 3 Collect sensor data

Drive the robot around with the keyboard as the turtlesim automatically collects pose and Lidar data. The teleop node can be stopped when you have collected enough data to generate a map of the entire area. Leave the `gmapping` process running.

```
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

### Step 4 Save Map

Create a directory in your custom turtlesim package for storing map files, then save the map in this directory with `map_saver`. Use the `-f` option to set the filename. Do not include a file extension on `<map_name>` in the commands below.
 
```
mkdir ~/catkin_ws/src/<package_name>/maps

cd ~/catkin_ws/src/<package_name>/maps

rosrun map_server map_saver -f <map_name>  
```

**For Example:** The two options below work on my system. 

_Option 1_ Use the absolute paths to the map file in new directory. 

```
mkdir ~/catkin_ws/src/turtlesim_control/maps

rosrun map_server map_saver -f ~/catkin_ws/src/turtlesim_control/maps/demo_world  
```

_Option 2_ Navigate to package directory and use relative paths. 

```
mkdir ~/catkin_ws/src/turtlesim_control/maps

cd ~/catkin_ws/src/turtlesim_control/maps

rosrun map_server map_saver -f demo_world  
```

Two new map files both with the same filename (`<map_name>.pgm` and `<map_name>.yaml`) will appear in the `maps` folder after Step 4. These map files can be moved or copied for for backup, but both files are required and the file names must match. After saving the map, stop the `gmapping` node.


## Part 3 - Navigate Virtual space:
Now that navigation is installed and there is a map saved to file, the robot can perform autonomous point to point navigation with dynamic obstacle avoidance using the map and RVIZ as the user interface. 

### Step 1 - Start the turtlebot3 simulator
``` 
  roslaunch turtlebot3_gazebo turtlebot3_world.launch
```

### Step 2 - Turn on navigation and RVIZ. 
Use the name of the map you created for the map_file option in the command. If you navigate to the package with the maps the absolute paths will work.

```
cd ~/catkin_ws/src/<package_name>

roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=maps/<map_name>.yaml

```

**For Example:** The three options below work on a my system.

_Option 1_ Navigate to the package directory and then use relative paths to the map files. 

```
cd ~/catkin_ws/src/turtlesim_control

roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=maps/demo_world.yaml

```

_Option 2_ Use the built-in ROS Change Directory tool `roscd` to quickly navigating to a ROS package on the system. This method is portable because it is not specfic to the location of the package containing the maps. 

```
roscd turtlesim_control

roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=maps/demo_world.yaml

```

_Option 3_ (**preffered method**) Use `find` to access the package containing the maps without changing the current directory. Use relative paths from the package directory in the find command.

```
 roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:='$(find turtlesim_control)/maps/demo_world.yaml'
```


The gazebo window will open containing your robot, and you will also see the rviz window open separately. Find and test the following features of navigation in RVIZ. 
	
- _Pose Estimate_ - Click and drag the direction to set the current pose estimate of the robot.
	
- _2D Nav Goal_ - Click and drag the direction to define a goal point for the robot in the map.

<img src="turtlebot3_rviz.png" alt="drawing" width="400"/>


Check the available topics with rostopic.
``` 
rostopic list
```

```
rosrun rqt_graph rqt_graph 
```

## Part 4 - Issues Loading Maps Files:

You may have run into an issue in which `turtlebot3_navigation` cannot load the map file. A typical error message is shown below, along with the preferred solution.

```
[ERROR] [1604667817.760623311]: Map server could not open /<map_name>.yaml.
```

### Step 1 
Check that the top line of `map.yaml` contains the name of the map with the `.pgm` extension. The path to the map file should not be included in this line. The file generated in the this tutorial is shown below.

```
image: demo_world.pgm
resolution: 0.050000
origin: [-10.000000, -10.000000, 0.000000]
negate: 0
occupied_thresh: 0.65
free_thresh: 0.196
```

### Step 2 
Check that `map.yaml` and `map.pgm` exist in a directory named maps/ in a package in the ROS workspace.

### Step 3 
Check that the workspace will compile by running
 `catkin_make` from the top of the workspace. You should see in the terminal output, that your package was successfully built. 

### Step 4

Use the `find` command to point the node to the maps directory of your package where the map files are stored. Repeat Part 3 with the addition of the `find` command in the map argument. 

Start the simulator
```
roslaunch turtlebot3_gazebo turtlebot3_world.launch
```
 
Turn on the navigation nodes and RVIZ. Use the name of the map you created, and it should be recogonized and displayed in RVIZ. 

```
roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:='$(find <package_name>)\maps\<map_name>.yaml'
```

### Tutorial Complete: 
After completing _Tutorial 6 - Turtlebot3 Simulator_, you are ready to learn about ... more ROS!
	
<img src="../../charlie_robot.jpeg" alt="drawing" width="400"/>

NOW, YOU KNOW ABOUT ROS! GOOD JOB!
	
	
		
