# generate URDF

The goal is to generate a URDF (Unified Robot Description Format) for a custom robot.

This tutorial begins by _reading_ the [ROS URDF tutorials](http://wiki.ros.org/urdf/Tutorials/Building%20a%20Movable%20Robot%20Model%20with%20URDF):

 - [Building a Visual Robot Model with URDF from Scratch](http://wiki.ros.org/urdf/Tutorials/Building%20a%20Visual%20Robot%20Model%20with%20URDF%20from%20Scratch)
 - [Building a Moveable Robot Model with URDF](http://wiki.ros.org/urdf/Tutorials/Building%20a%20Movable%20Robot%20Model%20with%20URDF)
 - [Create URDF for an Industrial Robot](http://wiki.ros.org/Industrial/Tutorials/Create%20a%20URDF%20for%20an%20Industrial%20Robot)



The R2D2 gripper shows how to use a .dae mesh file to define the geometry in the URDF file.

```
 166   <link name="left_gripper">
 167     <visual>
 168       <origin rpy="0.0 0 0" xyz="0 0 0"/>
 169       <geometry>
 170         <mesh filename="package://urdf_tutorial/meshes/l_finger.dae"/>
 171       </geometry>
 172     </visual>
 173   </link>
 ```




Also, there is a solidworks plugin solidworks_urdf_exporter for generating URDFs, but it does not appear to be supported in SW2021.




#### Create a workspace to test

```
mkdir -p catkin_make_ws/src
cd catkin_make_ws
catkin_make
```

#### Install neccesary ROS packages

```
sudo apt ros-melodic-joint-state-publisher ros-melodic-joint-state-publisher-gui
```

The package ros-melodic-urdf-tutorial came with `ros-melodic-desktop-full`, but there are some issue with that package. It needs to be updated for melodic (and the transition to Noetic). So clone it from `github/ros`, the default branch `ros1` should compile in the workspace.

```
cd catkin_make_ws/src
git clone https://github.com/ros/urdf_tutorial.git
cd ..
catkin_make
```

Now the example launch file should run without errors. 

```
roslaunch urdf_tutorial display.launch model:='$(find urdf_tutorial)/urdf/01-myfirst.urdf'
```
Try the different examples by changing the URDF file in the launch file.




All of the examples work except the last one which loads the gripper from a .dae + .tif. 



```
roslaunch urdf_tutorial display.launch model:=urdf/05-visual.urdf
 
 ...

TIFFReadDirectory: Warning, Sum of Photometric type-related color channels and ExtraSamples doesn't match SamplesPerPixel. Defining non-color channels as ExtraSamples..
TIFFFieldWithTag: Internal error, unknown tag 0x829a.
TIFFFieldWithTag: Internal error, unknown tag 0x829d.
TIFFFieldWithTag: Internal error, unknown tag 0x8822.
TIFFFieldWithTag: Internal error, unknown tag 0x8824.
TIFFFieldWithTag: Internal error, unknown tag 0x8827.
TIFFFieldWithTag: Internal error, unknown tag 0x8828.
```

Apparantly this error can be fixed by changing the TIFs to PNGs, but the DAEs need to be updated. This does not sound like what we want to do. 


Here is another tutorial [STL Mesh file for URDF Link](https://www.theconstructsim.com/ros-projects-my-robotic-manipulator-6-stl-mesh-file-for-urdf-link/) for _theconstruct_ which uses STL files directly. This is what we want to do. There is a video on that page that shows a useful example, but it is hard to follow without the source code. 

Clone the source code from bitbucket to use as another example.

```
cd catkin_make_ws/src
git clone https://bitbucket.org/theconstructcore/my-robotic-manipulator/src/master/
```

Apparantley, cloning from bitbucket acts differently. The directory appears as `master`. Re-name this to avoid confusion and to match the tutorial.

```
cp -r master mrm_description
cd ..
catkin_make
```

The examples seem to run, but the simulation environment does not appeaer to be setup. This may be because that tutorial uses a browser-based environment (non install).

So intead of using that package, borrow some code from it. The following files are used to load a mesh file from STL into the URDF.


`urdf/me4140-example.urdf`


Run the following command to view the model in rviz

```
roslaunch urdf_tutorial display.launch model:='$(find urdf_tutorial)/urdf/me4140-example.urdf'
```

Ah, now we have the classic tf error. No reference for the base frame. We need to add a node to setup the fixed transforms. 

OK, my tf_broadcaster.cpp would not compile in that package, so I made a package from scratch. I copied in the neccesary folders from urdf_tutorial.. I think


Now, use this launch instead.

```
roslaunch generate_urdf display.launch model:='$(find generate_urdf)/urdf/me4140-example.urdf'
```

OK, this new package works. The tf error is still there because we have not the tf_broadcaster node. Add this to the display.launch

Okie Dokie! The tf error is fixed. This is progress. 