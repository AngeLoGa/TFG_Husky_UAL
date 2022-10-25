# TFG_Husky_UAL
This is a public repository where there are all the files required for the simulation of my final degree project, as well as some useful instructions to use the package

This is a trial for the foxy branch

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
ros2 launch agricobiot_config online_async_launch.py
```

Terminal C:

```
ros2 launch agricobiot_config view_robot_launch.py
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

If that's not your case, there is a launch named "map_saver.launch.py inside the package agricobiot_config. You will need to launch it and then call a service as the previous case in order to save the map. The only difference between them is that this last one uses a config file where the timeout is set to 5000 (which is more than enough to compute a big map as the one of the greenhouse or even bigger). In order to do it, you can type as follow:

Terminal E:

```
ros2 launch agricobiot_config map_saver.launch.py
```

Terminal F (remember to be in the directory where you want your map saved):

```
ros2 service call /map_saver/save_map nav2_msgs/srv/SaveMap "{map_topic: map, map_url: <map name>, image_format: pgm, map_mode: trinary, free_thresh: 0.25, occupied_thresh: 0.65}"
```
### Localization
In order to localize the robot, we need to provide the map generated using SLAM using map_server from nav2_map_server, and launch the amcl node configured as the previous ones, config file in the config directory in agricobiot_config. These two are supervised by a lifecycle_server (implementation of ROS2).

Terminal A is the same as previous

Terminal B

```
ros2  agricobiot_config view_robot_launch.py
```
You can change the config of rviz and use the one for localization, named  "localizer_config.rviz", or add the map inserting the map topic on it.

Terminal C

```
ros2  agricobiot_config localization.launch.py
```
Now that all is set, you should see in the Terminal C a message like "AMCL cannot publish a pose or update the transform..." If this happens, don't worry, it is okey, you just need to set the estimation pose on RVIZ. Make sure the global frame is set to map, if not, you will see in the AMCL terminal that appears a message saying that the pose is being rejected because it needs to be publish over the frame map.

In order to make the localization, we need to move the robot after setting the initial pose, with teleop as was explained in the previous sections.

:exclamation::exclamation:Problems with this launch: As you can see, AMCL publish some topics like, particle_cloud and amcl_pose. These ones in RVIZ should look something like this: 
![951536fa-4492-4696-915a-58a6e4074b0c](https://user-images.githubusercontent.com/98213868/192593694-0d5477c5-47db-4796-b37c-d5b4df85efbd.png)

But in this case, you will see that it doesn't show any of them. The only thing I have being able to get is to see the amcl_pose as a small circle under the husky, which should be way bigger if it isn't located. So I think the problem resides in making the amcl work properly, feel free to touch the config files and make questions about it.

### Navigation
Everything is set up for the navigation stuff, but the AMCL problem needs to be solved before using this launch. But in resume, it is the localization launch adding the path_planner server, the bt_navigator and the controller for the autonomous movement. Lastly, the lifecycle manager has been updated in order that it manages also the new nodes.




## Others

Convert a ROS1 .bag into an MRPT .rawlog file:

    rosbag2rawlog -w -f world \
        -c agricobiot_config/config/config_rosbag2rawlog.yml \
        -o output.rawlog \
        INPUT.bag

Then open with `RawLogViewer *.rawlog`.


