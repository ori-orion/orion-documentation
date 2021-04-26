# HSR mapping

## Creating Occupancy Map
SSH into the HSR and run the following commands:
1. `source /opt/ros/kinetic/setup.bash`
2. `rosnode kill /pose_integrator`
3. 
	a. `rosrun hector_mapping hector_mapping _map_size:=2048 _map_resolution:=0.05 _pub_map_odom_transform:=true _scan_topic:=/hsrb/base_scan _use_tf_scan_transformation:=false _map_update_angle_thresh:=2.0 _map_update_distance_thresh:=0.10 _scan_subscriber_queue_size:=1 _update_factor_free:=0.39 _update_factor_occupied:=0.85 _base_frame:=base_range_sensor_link`
	b. `Alternatively (and recommended): $ rosrun  gmapping slam_gmapping scan:=/hsrb/base_scan`
4. `On your laptop, do the following:`
	a. `hsrb_mode`
	b. `rviz -d <config_file>`
5. Start moving the robot around, either using the joystick as described in the previous document or by running the following commands in another sshed terminal:
	a. `hsrb_mode`, then `rqt`.
	b. When rqt starts running, go to plugin > robot tools > robot steering. Make sure the topic at the top of the window is /hsrb/command_velocity instead of /cmd_vel
6. When moving the robot around you can now see the map being built up in Rviz.
7. When done, run `rosrun map_server map_saver`. DO NOT terminate `hector_mapping`/`gmapping` until after this is done! Afterwards you can terminate.
8. Create a directory in `/etc/opt/tmc/robot/conf.d` to store your two map files. You will likely need sudo for operations in `/etc`.
9. Crop the map using either `rosrun topological_utils crop_map.py map.yaml` or `rosrun map_server crop_map map.yaml`. This creates `cropped.yaml` and `cropped.pgm`.
10. In `cropped.yaml`, change the reference to `cropped.pgm` to `map.pgm`.
11. Move `cropped.pgm` and `cropped.yaml` to the directory we created in `/etc`, renaming them `map.pgm` and `map.yaml` in the process. Important: They MUST be called `map.pgm` and `map.yaml`.
12. Go to `/etc/opt/tmc/robot/docker.hsrb.user` and set `MAP_PATH` to: `MAP_PATH=/etc/opt/tmc/robot/conf.d/<new_directory>`.
13. Run `sudo systemctl restart docker.hsrb.robot.service`
14. Restart the robot's computer and the map should appear when you run rviz.


## Create Topological Map:

1. To ensure the robot updates its map information, you may want to press its emergency stop button to stop/start all the nodes again.
2. SSH into HSR and run the following command: `roslaunch mongodb_store mongodb_store.launch db_path:=/home/administrator/db`.
3. SSH into HSR in another terminal and run the command: `rosrun topological_utils insert_empty_map.py name_of_topological_map` (This will have inserted an empty map into the mongo database).

## Edit the Topological Map:

1. Kill any mongo/topological_navigation instances already running.
2. Run: `roslaunch topological_rviz_tools strands_rviz_topmap.launch standalone:=true topmap:=<name of map> db_path:=/home/administrator/db/ map:=/etc/opt/tmc/robot/conf.d/<map_dir>/map.yaml`
3. On the laptop locally, do:
	a. `hsrb_mode`
	b. `rviz -d `rospack find topological_rviz_tools`/conf/topological_rviz_tools.rviz`.
4. You can now add nodes in rviz and make edges between them.
	a. Add some nodes. They can be rotated by clicking and moving on the blue circle around the node. 
	b. Delete the temp node once some nodes have been added.
	c. Make sure to rotate a node once you add it or it won't be added correctly.
	d. Add edges between your nodes.
	e. Set the action correctly on the edges. If an edge doesn't go through a door, set the action to `move_base/move`, if it does, set it to `door_open_and_move_base`. 
5. Try running everything as per usual, making sure to change your map name when starting topological navigation.
6. You will also have to run: `rosrun orion_door_pass door_open_and_move_base.py`.
7. If the map doesn't show up properly in rviz after doing all of this, you may want to completely restart the robot, and then try again.

## Add labels to nodes on the topological map:

This would be necessary if you wanted to label nodes with rooms for example (e.g. kitchen). Do the following:

1. ```rosservice call /topological_map_manager/add_tag_to_node "tag: 'Kitchen'
node:
- 'WayPoint3'"``` to set the labels.
2. `rosservice call /topological_map_manager/get_tagged_nodes "tag: 'Kitchen'"` to get the nodes that have this particular label.
