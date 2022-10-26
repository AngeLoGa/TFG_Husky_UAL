# TFG_Husky_UAL
This is a public repository which stores all the files required for the simulation of my final degree project, as well as some useful instructions to use the package. Please feel free to make issues regardind doubts whether if is for ROS2 issues, as well as this package.

I publish all my project not also to show it to everyone, but also to improve this package and keep learning and improving!

## :pencil2: Authors :pencil2:
- Ángel López Gázquez, as the main developer.
- Francisco José Mañas Alvarez, co-tutor of the final degree project, as well as contributor.
- Jose Luis Blanco Claraco, contributor and overseer of the package.

## :raised_hands: Credits :raised_hands:
I have to credit my second tutor of the project, Francisco Jose Mañas Alvarez, who most helped me in developing this package, as well as being my master. Also thanks to Jose Luis Blanco, for being interested in the package even without being part of it, and helping me in the task of learning ROS2, C++, Python, Debug, MVSLAM, do I need to follow? (joke JL hehe, I think JC won't let me be that personal sadly).

## DISCLAIMER :exclamation:
This project was developed using Ubuntu 20.04 with ROS2 Foxy Devel. I may update it with the new Humble edition. Please stay tunned and feel free to contact me via Linkedin or via email if you are interested in it.

## Install :book:

### First requirements 📋
##### ROS2
[Foxy Desktop](https://docs.ros.org/en/galactic/Installation/Ubuntu-Install-Debians.html)

##### Packages
```
sudo apt-get install ros-foxy-hardware-interface 
sudo apt-get install ros-foxy-controller-manager ros-foxy-gazebo-ros2-control ros-foxy-xacro ros-foxy-hardware-interface ros-foxy-gazebo-plugins ros-foxy-gazebo-msgs ros-foxy-gazebo-ros ros-foxy-gazebo-ros2-control-demos ros-foxy-slam-toolbox ros-foxy-navigation2 ros-foxy-nav2-bringup
``` 

### For Ubuntu 20.04
```
cd /<your_workspace>/src
git clone --recurse-submodules -b foxy-dev https://github.com/AngeLoGa/TFG_Husky_UAL.git
git clone -b foxy-devel https://github.com/husky/husky.git
cd husky
git checkout 07684eb894c247aa0e046952cb0dc890d2944257
cd ../..
rosdep install --from-paths src --ignore-src -r -y
colcon build
```

## First simulation Greenhouse
Make sure you have installed from source all the packages as shown in the section "Packages" above.
Also, remember to source in each terminal your setup.bash from your workspace before using any commands provided next (from <your_workspace>):


Terminal A:

```
. install/setup.bash
```

```
ros2 launch husky_ual husky_simulation.launch.py
```

You will be able to see the robot in the Greenhouse at Gazebo but also in RVIZ. If you are interested in what is happening behind all this nodes, there is an urdf which is published in order to see the robot in RVIZ, while the model in Gazebo comes from a SDF files. This is normally used in robotics since urdfs sometimes don't work properly in Gazebo. You may want to add whichever topic you want to see in RVIZ.

In order to move the robot, use the next command line:

Terminal B:

```
. install/setup.bash
```

```
ros2 run teleop_twist_keyboard teleop_twist_keyboard --prefixros-args --remap cmd_vel:=/cmd_vel
```

Now you can follow the instructions seen in the terminal so you can move the robot through the greenhouse.

### Mapping
In order to do SLAM with this package, make sure you installed from source Slam-Toolbox. We will launch husky_navigation, which launches nav2_bringup, with the parameters we provide inside the config directory. You may want to twist some parameters so you can see how it affects the navigation. Please feel free to try anything you want, remember this is a package provided to be a tutorial for starters in ROS2, while showing the project of the TFG.

Terminal A:

```
ros2 launch husky_ual husky_simulation.py
```

Terminal B:

```
ros2 launch husky_ual online_async_launch.py
```

You might want to change the configuration, you can do it adding the map topic using the panels of RVIZ, or you can change the config charging the one provided inside the package agricobiot_config in the directory rviz, named mapping_config.rviz [I'm working at using an argument for the RVIZ_config file or rather use another view_robot_launch just for mapping issues]

You should see something like this:

![Screenshot from 2022-09-27 13-05-13](https://user-images.githubusercontent.com/98213868/192509587-00c685f1-e2f6-40a4-8a98-423993705ee3.png)

As you can see there is a bit of map generated, that is a good signal that everything is fine (don't worry if the terminals B and C pop messages that are being filtered (it is because the sensor rate is way bigger than the map rate being refreshed). If you don't see the map, you might want to press the button "Reset" from RVIZ at the left side, ir order to update RVIZ, just in case the online_async was not loaded before RVIZ started. Now you can start to move your robot and map the environment

Terminal D:

```
ros2 run teleop_twist_keyboard teleop_twist_keyboard --prefixros-args --remap cmd_vel:=/cmd_vel
```

By the time you have finished mapping, you might won't be able to use the typical service from map_server map_saver because there was no fix for an issue regarding time limitation for it to compute the map in the distribution Foxy. So, you might want to try first if the map_saver works for the map you made, but before you use the command, make sure you launch it inside the directory where you want to save the map (there is a maps carpet inside the package agricobiot_config where you might want to save your maps):

Terminal E:

```
ros2 run nav2_map_server map_saver_cli -f <name of the map>
```
You will see two new records, a .png and a .yaml If you see both and you can see in the .png your map you are done!

If that's not your case, there is a launch named "map_saver.launch.py inside the package map_saver. You will need to launch it and then call a service as the previous case in order to save the map. The only difference between them is that this last one uses a config file where the timeout is set to 5000 (which is more than enough to compute a big map as the one of the greenhouse or even bigger). In order to do it, you can type as follow:

Terminal E:

```
ros2 launch map_saver saver.launch.py
```

Terminal F (remember to be in the directory where you want your map saved):

```
ros2 service call /map_saver/save_map nav2_msgs/srv/SaveMap "{map_topic: map, map_url: <map name>, image_format: pgm, map_mode: trinary, free_thresh: 0.25, occupied_thresh: 0.65}"
```
This should let you save your map without problems.

### Navigation
In order to use the navigation launch, please follow the next instructions.

Terminal A:

```
ros2 launch husky_ual husky_navigation.py
```

You should have Gazebo and RVIZ, like the husky_simulation launch file, but this time two things changed. First, the robot in RVIZ doesn't pop u, but there is already a map of the greenhouse. You can't see the urdf robot model because you need to set its position, as shown in the memory.

Please try to be the most accourate possible setting the initial pose of the robot, remember that the localization package is not working properly, so the robot won't be able to localize for its own.

By the time you set the pose, the robot will spawn in that position, and all the navigation stack will start to work, showing global costmaps, local costmaps, etc.

Now you can use whether the Nav2 Goal tool, or the Nav2Plugin in order to set some waypoints, to make a custom path.

