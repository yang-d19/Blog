---
title: Ego Planner Reading Note
typora-root-url: ./
categories: [Technology]
tags: [Ego Planner]
date: 2023-10-03 15:46:57
---

This is the reading note for [ZJU-FAST-Lab/ego-planner-swarm: An efficient single/multi-agent trajectory planner for multicopters. (github.com)](https://github.com/ZJU-FAST-Lab/ego-planner-swarm)

This code repo is so complex that I can only go through a small part of it. In this reading note, I start from the process of launching the file: single_run_in_sim.launch. That is, what nodes does single drone in simulation environment uses and how do these nodes communicate with each other.

I also include detailed analysis of some important source file. For example, what are their core functions and what is the calling process of them.

<!-- more -->

# [0] Draw node relationship graph

```
rosrun rqt_graph
```

Must be executed after launching ros shell script. It can draw the relation graph of all nodes and topics that are used.

------

# [1] launch rviz with config file

from planner/plan_manage_launch/default.rviz

the config file is modified every time you change config in rviz.

------

# [2] launch single_run_in_sim.launch

+   Sub means subscribe
+   Pub means publish
+   Arrows → and ← represent the mapping relation of topics or nodes

>For some important topics, I labeled the topic message type, the sender or receiver functions and the calling frequency. I drew some nodes and topics relation graphs with [draw.io](http://draw.io) to make it more clearer.


## 1 launch node [map_generator/random_forest]

in source file *planner/map_generator/random_forest_sensing.cpp*

### Sub1 topic odometry

unused

### Pub1 topic “/map_generator/global_cloud”

through publisher _all_map_pub in function `pubSensedPoints()`

data source is cloudMap, which is generated in function `RandomMapGenerateCylinder()`,

which is called once in `main()`

>   There exist another similar function RandomMapGenerate(). The only difference between them is the original one generates square colomn instead of cylinder ones. They only differ in one line! (Why not merge them into one function?)

### Pub2 topic “/map_generator/local_cloud”

unused

>   The topic is never used as relevant codes in function pubSensedPoints() is behind a return statement. What’s more, the relevant codes requires odom message which comes from “odometry” topic, which is not provided to the random_forest node.

### Node-Topic Graph

<img src="/Ego-Planner-Reading-Note/node-random_forest.png" alt="node-random_forest" style="zoom:30%;" />

## 2 include xml file advanced_param.xml

in which contains the following nodes:

## 2a launch node ego_planner/ego_planner_node as drone\_$(arg drone_id)\_ego_planner_node → [drone_0_ego_planner_node]

source files: ( all in planner/plan_manage/src/ folder)

-   ego_planner_node.cpp (Just the launcher of fsm)
-   ego_replan_fsm.cpp (Contains the finite state machine)
-   planner_manager.cpp (Ego planner manager, used by fsm)

>   output is sent to screen

:bulb: This is the core node of ego planner. It has many dependencies as shown below. (in its CMakeLists.txt)

```cpp
catkin_package(
	INCLUDE_DIRS include
	LIBRARIES ego_planner
	CATKIN_DEPENDS plan_env path_searching bspline_opt traj_utils 
)
```

**brief summary of the above packages**

+   **plan_env**: map reconstruction 

+   **path_searching**: dynamic A\* search

+   **bspline_opt**: bspline optimization

+   **traj_utils**: PolynomialTraj related function and visualization

### Sub1 topic odom_world ← /drone\_\$(arg drone_id)\_\$(arg odometry_topic) ← “drone_0_visual_slam/odom”

ego_replan_fsm.cpp

**<nav_msgs::OdometryConstPtr>**

— *remap in advanced_param.launch*

with callback function `odometryCallback()`

which extract the received message to `odom_pos_` & `odom_vel_` & `odom_orient`_, then set `have_odom_` true

### Sub2 topic “/drone\_$(drone_id - 1)\_planning/swarm_trajs”

>   ego_replan_fsm.cpp

no use

only subscribe to this topic when there are swarm, not single drone

>   4 topics below are all contained in *planner/src/grid_map.cpp*
>
>   +   grid_map/depth & grid_map/cloud -- provide topics about the surrounded environment
>   +   grid_map/odom & grid_map/pose -- provide topics about their own position
>
>   In order to reconstruct the map, we need at least one topic from each type
>
>   
>
>   Under simulation launch file，we use cloud & odom topics；
>
>   Under realworld launch file，we use depth & odom topics
>
>   
>
>   In order to support RGB map reconstruction，we need to adjust data structures used by cloud & depth topic

### Sub3 topic grid_map/depth ← /drone\_\$(arg drone_id)\_\$(arg depth_topic) ← "drone_0_pcl_render_node/depth”

:bulb: Depth topic is only used in **realworld exploration**

**<sensor_msgs::Image>**

— *remap in advanced_param.launch & singhle_run_in_sim.launch*

by subscriber `depth_sub_` in file *grid_map.cpp*

*IF* pose_type_ == POSE_STAMPED: **[← Chose in simlulation]**

  Sync depth with pose in `sync_image_pose_ `with callback function `depthPoseCallback()`

*ELIF* pose_type_ == ODOMETRY: **[← Chose in realworld]**

  Sync depth with `odom `in `sync_image_odom_ `with callback function `depthOdomCallback()`

**How to figure out the correspondence**

+   grid_map.cpp has `node_.param("grid_map/pose_type", mp_.pose_type_, 1)`;

    -   advanced_param.xml has` <param name="grid_map/pose_type" value="1"/>`

    -   advanced_param_exp.xml has`<param name="grid_map/pose_type" value="2"/>`


+   *grid_map.h* has `enum { POSE_STAMPED = 1, ODOMETRY = 2, INVALID_IDX = -10000 }`;

However, in realworld exploration, we use grid_map/depth instead of grid_map/cloud, so the two callback functions above are never called in realworld exploration process.

### Sub4 topic grid_map/cloud ← /drone\_\$(arg drone_id)\_\$(arg cloud_topic) ← “drone_0_pcl_render_node/cloud”

:bulb: Cloud topic is only used in **simulation**

**<sensor_msgs::PointCloud2>**

— *remap in advanced_param.launch & singhle_run_in_sim.launch*

by subscriber `indep_cloud_sub_` in file *grid_map.cpp*

with callback function `GridMap::cloudCallback()`

**cloudCallback() function conclusion：**

+   Change messages of type `sensor_msgs::PointCloud2` to latest_cloud of type `pcl::PointXYZRGB` (In order ot support RGB map reconstruction, I change `PointXYZ `to `PointXYZRGB `type)

+   Check if odom message is updated、if latest_cloud is null, if camera_pos is null

+   Iterate through all point in `latest_cloud`，inflate each point, set `md_.occupancy_buffer_inflate_[idx_inf] = rgb` (Originally it is 1，I changed it to `rgb `value)

+   `idx_inf `is calculated by coordinate of `point `，I wonder why the author not use 3-d array, it is more intuitive

### Sub5 topic grid_map/odom ← /drone\_\$(arg drone_id)\_\$(arg odometry_topic) ← “drone_0_visual_slam/odom”

**<nav_msgs::Odometry>**

— *remap in advanced_param.launch & singhle_run_in_sim.launch*

by subscriber odom_sub_ without callback function (Not separate callback functions，because `odom_sub_ `and  `depth_sub_ `is merged into `sync_image_odom_ `and callback together)

or `indep_odom_sub_ `with callback function `GridMap::odomCallback()`

`odomCallback()` simply extract `camera_pos `from `nav_msgs::OdometryConstPtr`and set `has_odom_ = true`

### Sub6 topic grid_map/pose

TO DO

:exclamation: In simulation, map reconstruction module (mainly in grid_map.cpp) uses “grid_map/cloud”. In realworld, however, it uses “grid_map/depth” . Why simulation cannot use “grid_map/depth” ???

### Sub7 topic planning/broadcast_bspline_to_planner ← "/broadcast_bspline”

ego_replan_fsm.cpp

**<traj_utils::BsplinePtr>**

— *remap in advanced_param.launch*

with callback function `EGOReplanFSM::BroadcastBsplineCallback()`

**Brief Summary:** TO DO

### Sub8 topic “/move_base_simple/goal” or "/traj_start_trigger”

ego_replan_fsm.cpp

**How I confirm the topic is the latter:**

+   subscribe to which topic depends on value of target_type_, the value is got from:
+   ← `nh.param("fsm/flight_type", target_type_, -1);` (*ego_planner_fsm.cpp*)
+   ← `<param name="fsm/flight_type" value="$(arg flight_type)" type="int"/>` (*advanced_param.xml*)
+   ← `<arg name="flight_type" value="2" />` (*single_run_in_sim.launch*)

which means `target_type_ == TARGET_TYPE::PRESET_TARGET`, so the real subscription is "/traj_start_trigger”

**<geometry_msgs::PoseStampedPtr>**

with callback function `EGOReplanFSM::triggerCallback()`

which set` have_trigger_ = true`, and initialize `init_pt_`

### Pub1 topic grid_map/occupancy

no use

### Pub2 topic grid_map/occupancy_inflate → “/drone_0_ego_planner_node/grid_map/occupancy_inflate”

**<sensor_msgs::PointCloud2>**

*— only convert from local node namespace to global*

:sunny: This is the reconstructed map that is shown in rviz

`visCallback()` includes `publishMapInflate()` and `publishMap()`;

which publish grid_map/occupancy and grid/occupancy_inflate, only the former one is used

### Pub3 topic “/drone\_\$(drone_id)\_planning/swarm_trajs”

*ego_replan_fsm.cpp*

no use

### Pub4 topic planning/broadcast_bspline_from_planner → "/broadcast_bspline”

*ego_replan_fsm.cpp*

**<traj_utils::Bspline>**

— *remap in advanced_param.launch*

by publisher `broadcast_bspline_pub_ `in function `EGOReplanFSM::publishSwarmTrajs()`

which has no specific frequency, but is called in FSM

### Pub5 topic "planning/bspline”

TO DO

### Pub6 topic "planning/data_display”

TO DO

## 2.3 launch node ego_planner/traj_server as [drone\_\$(arg drone_id)\_traj_server] → [drone_0_traj_server]

in source file *planner/plan_manage/traj_server.cpp*

>   output is send to screen

### Sub1 topic planning/bspline ← drone\_\$(arg drone_id)\_planning/bspline ← “drone_0_planning/bspline”

**<traj_utils::BsplineConstPtr>**

— *remap in single_run_in_sim.launch*

with callback function `bsplineCallback()`

### Pub1 topic /position_cmd → drone\_\$(arg drone_id)\_planning/pos_cmd → “drone_0_planning/pos_cmd"

**<quadrotor_msgs::PositionCommand>**

— *remap in single_run_in_sim.launch*

through publisher `pos_cmd_pub `in function `cmdCallback()`, which is callback function of Timer `cmd_timer`, frequency 100Hz

data source is `cmd`, generated in `cmdCallback()`

**Brief Summary of cmdCallback()**:

-   check `receive_traj_ `to see if received bspline messages, variable defined in `bsplineCallback()`
-   if` curr time < traj duration`
    -   then calculate` cmd.pos & vel & acc` by DeBoor Algorithm `evaluateDeBoorT()`. And calculate yaw by `calculate_yaw()` (a self-defined function)
    -   else, hover at the end point of the trajectory, `cmd.vel & acc = 0`, `yaw `keep unchanged
-   combine variables above into `cmd`, and publish it through `pos_cmd_pub `to /position_cmd

### Node-Topic Graph

<img src="/Ego-Planner-Reading-Note/node-traj_server.png" alt="node-traj_server" style="zoom:30%;" />

💡 B-spline curve is shaped by control points and basic functions. Basic functions are defiend by knot vector and curve order k, which influence the continuity and shape of the curve.

[1.4.2 B-spline curve (mit.edu)](https://web.mit.edu/hyperbook/Patrikalakis-Maekawa-Cho/node17.html)

## 2.4 launch xml file ego_planner/launch/simulator.xml

in which contains the following nodes:

## 2.4a launch node poscmd_2_odom/poscmd_2_odom as drone\_\$(arg drone_id)\_poscmd_2_odom → [drone_0_poscmd_2_odom]

node source file is *uav_simulator/fake_drone/poscmd_2_odom.cpp*

### Sub1 topic command ← drone\_\$(arg drone_id)\_planning/pos_cmd ← “drone_0_planning/pos_cmd”

**<quadrotor_msgs::PositionCommand>**

— *remap in simulator.xml*

with callback function `rcvPosCmdCallBack()`, which is called by `ros::spinOnce()` at a frequency of 100Hz.

### Pub1 topic odometry → drone\_\$(arg drone_id)\_\$(arg odometry_topic) → “drone_0_visual_slam/odom”

**<nav_msgs::Odometry>**

— *remap in simulator.xml*

through publisher _odom_pub in function pubOdom(), which runs in a while loop at a frequency of 100Hz.

### Node-Topic Graph

<img src="/Ego-Planner-Reading-Note/node-poscmd.png" alt="node-poscmd" style="zoom:33%;" />

## 2.4b launch node odom_visualization/odom_visualization as drone\_\$(arg drone_id)\_odom_visualization → [drone_0_odom_visualization]

node source file is uav/simulator/Utils/odom_visualization/odom_visualization.cpp

### Sub1 topic odom ← drone\_\$(arg drone_id)\_visual_slam/odom ← “drone_0_visual_slam/odom”

with callback function odom_callback() called by ros::spin()

💡 Difference between ros::spin() & ros::spinOnce() and usage of ros::Rate(), see at [Callbacks and Spinning - ROS Wiki](https://wiki.ros.org/roscpp/Overview/Callbacks and Spinning)

### Sub2 topic cmd

no use

### Pub1 topic robot → “drone_0_odom_visualization/robot”

**<visualization_msgs::Marker>**

by publisher meshPub, in function `odom_callback()` called by `ros::spin()`

💡 If topic begin with “/”, then it’s a global topic, other nodes can subscribe to it by its original name. Otherwise, it’s a local variable, other nodes need to subscribe to “<node_name>/<topic_name>”

❓ **Need Verification**: It seems that in xml and launch files, every topic name without a “~” prefix is regarded as global topic.

### Pub2 topic path → “drone_0_odom_visualization/path”

**<nav_msgs::Path>**

by publisher pathPub, in function `odom_callback()` called by `ros::spin()`

### Pub3…N topic pose, velocity, covariance…

no use

💡 If a published topic is not subscribed by any node, it will not be visible on the rqt_graph. However, if a topic is subscribed by rviz node, although there is not a connecting line from topic to rviz, the topic will be visible.

❓ The rviz node in rqt_graph is shown as n__rviz and only connect a line to ‘/tf’ (cuboid like stuff). Don’t know why.

### Pub4 tf::Transform “/tf”

**[tf::StampedTransform](tf::StampedTransform)**

by `tf::TransformBroadcaster* broadcaster`, in function `odom_callback() `called by `ros::spin()`

from *line 363 of odom_visualization.cpp*

🔆 This is not a common ros topic, but do provide transform information to rviz. Need to learn more if time allows.

<img src="/Ego-Planner-Reading-Note/node-odom_visualization.png" alt="node-odom_visualization" style="zoom:30%;" />

## 2.4c launch node local_sensing_node/pcl_render_node as drone\_\$(arg drone_id)\_pcl_render_node → [drone_0_pcl_render_node]

node source file depends on whether to enable cuda,

if so, use *uav_simulator/local_sensing/pcl_render_node.cpp*;

else use uav_simulator/local_sensing/pointcloud_render_node.cpp

I change the two file names to *pcl_render_node_cuda.cpp* and *pcl_render_node_cpu.cpp* in [yang-d19/ego-planner-pro: Based on ego planner, enabled colored maping (github.com)](https://github.com/yang-d19/ego-planner-pro)

>   Send output to screen for review, by specify [output = “screen”]
