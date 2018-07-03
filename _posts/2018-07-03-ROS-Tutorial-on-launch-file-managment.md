# How to Launch ROS nodes properly

Launch ROS nodes is about start running actual processes on machines. 

This tutorial assume that readers have basic ideas of how ROS nodes are origanized, suck as master server, nodes,  publisher/subscriber, message and topic. You can aquire these ideas from the beginner tutorial at ROS wiki. 

This tutorial is about the roslaunch tool, using our Robocon code as an example. On our robot, we use roslaunch tool to start the robot. 

## why do we want roslaunch?

There are many ways to start a set of ROS nodes into running. The naive way to run nodes is starting master with 'roscore' and follow by a series of 'rosrun'. The commands will look like this:

    roscore # start the ROS master server
    rosrun package0 node0 # start node0 from package0
    rosrun package1 node1 # start node1 from package1
    ...

The above approach of starting application is useful when testing one or two nodes. When we try to build applications with several nodes, obviously, it's tedious to do such kind of work every time you want to run your robot. Fortunatly, ROS provide the tool 'roslaunch' to help you to start many nodes in one instruction. roslaunch relies on launch files(.launch) for node management. You can find a brief introduction to launch files on ROS wiki (
    [roslaunch on ROS wiki](http://wiki.ros.org/roslaunch/)
)

In a word, using roslaunch we can start a bunch of node with one shoot.

    roslaunch package_name launch_file_name.launch

What I'm interested in talking about is launching medium to large applications, for example robots having some 30 to 40 different nodes. I will take the launch files I wrote for Robocon in summer 2018 as an example, to explain some basic ideas and tools which are useful in launching big applications. (
    [launch files for Robocon2018](https://github.com/yxiao1996/RoboTop/tree/master/catkin_ws/src/robocon/launch)
)

## the basic files structure for launching

To make this tutorial self-contained, I'm going to briefly introduce the basic format of writing launch files. Here is an simple example we used to launch the circle detector and roi filter.

    <launch>
        <include file="$(find usb_cam)/launch/usb_cam-test.launch">
        </include>
        <rosparam command="load" file="$(find robocon)/config/baseline/train_detect/data.yaml"/>
        
        <node name="circle_detector" pkg="circle_detector" type="circle_detector_node.py" output="screen">
        </node>
        
        <node name="roi_filter" pkg="roi_filter" type="roi_filter_node.py" output="screen">
            <remap from="~set_ref" to="circle_detector/set_ref"/>
            <remap from="~roi" to="circle_detector/roi"/>
        </node>
    </launch>

Let us go through this piece of code tag by tag:

    <launch>
    ...
    </launch>

This pair of tag indicates that this is a launch file for roslaunch

    <inlcude file="$(find usb_cam)/launch/usb_cam-test.launch">
    ...
    </include>

This include tag load a existing launch file "usb_cam-test.launch", which starts a web camera capturing.

    <node name="circle_detector" pkg="circle_detector" type="circle_detector_node.py" output="screen">
    </node>

This node tag define the node we want to start be roslaunch. The node will be named as "circel_detector", and roslaunch will try to find executable code "circle_detector_node.py" at package "circle_detector".

## two-layer launch structure for big application

To run many nodes freely on a single machine, we adopted a two-layer launch file structure. The bottom-layer acts like an infrastructure, encapsulating all the available nodes and necessary message connections among nodes. Then, it provide switches for top-layer launch files to choose from. The bottom-layer launch file in our project is called '**master.launch**'. There are multiple top-layer launch files in our project, pretty much take the same idea of switching the desire nodes into usage. 

For example, part of the **master.launch** looks like this:
    
    ...
    <arg name="encoder" value="true"/>
    ...
    <!-- Start Serial Port Encoder -->
    <group if="$(arg encoder)">
        <node ns="$(arg veh)" pkg="serial_encoder" type="serial_encoder_node.py" name="serial_encoder_node" output="screen">
            <rosparam command="load" file="$(find robocon)/config/$(arg config)/serial_encoder/serial_encoder_node/$(arg param_file_name).yaml"/>
            <param name="protocol_auto_flag" type="bool" value="$(arg protocol_auto_flag)"/>
            <remap from="/Robo/serial_encoder_node/joy_auto" to="/Robo/joy_mapper_node/joy_auto"/>
            <remap from="/Robo/serial_encoder_node/joy_remote" to="/Robo/joy_mapper_node/joy_remote"/>
            <remap from="/Robo/serial_encoder_node/odo_data" to="/Robo/odo_control_node/twist2d"/>
            <remap from="/Robo/serial_encoder_node/pid_data" to="/Robo/pid_control_node/twist2d"/>
            <remap from="/Robo/serial_encoder_node/move_data" to="/Robo/move_planner_node/move"/>
        </node>
    </group>

This part of code defines the serial port encoder node on our robot. Through this node, the computer transfer control commands to micro-controller by serial port, and thereby control the action of the robot. 

Let us metion some pairs of tags: 
* group: encapsulate the encoder node with a switch called '**encoder**', which is a parameter defined previously.
* node: define the node.
* rosparam: load parameters from a specific .yaml file.
* remap: define the connection between encoder node and other node. 

After encapsulation, we can choose whether to use this node by toggling the '**encoder**' switch in top-layer launch file like this: 

    <include file="$(find robocon)/launch/master.launch">
        ...
        <arg name="encoder" value="true"/>
	    ...
    </include>

Similar encapsulation is applied to all nodes. 

## topic remapping among nodes

An important part of work in defining launch file is to make sure that nodes communicate properly with each other. ROS adopting a Publisher/Subscriber model, where there is a publisher only send out message, and a subscriber only receive message. That is to say, defining node communication is connecting nodes with the correct topic from both publisher and subscriber sides. 

In this section, I will take an example pair of node in our Robocon project to demonstrate how to connect topic between two nodes properly. The nodes I'm going to use is circle detector node and roi filter node. Here I think I should introduce their input/output relationship: the circle detector node take camera image as input, and output the cropped image according to the most possible circle in the image; then the cropped image is send to roi filter node by topic. 

    <!-- Start ROI -->
    <group if="$(arg roi)">
        <node ns="$(arg veh)" name="circle_detector" pkg="circle_detector" type="circle_detector_node.py" output="screen">
            <remap from="~image_compressed" to="usb_cam1/image_raw/compressed"/>
        </node>
  
        <node ns="$(arg veh)" name="roi_filter" pkg="roi_filter" type="roi_filter_node.py" output="screen">
            <remap from="~set_ref" to="move_planner_node/set_ref"/>
            <remap from="~roi" to="circle_detector/roi"/>
        </node>
    </group>

Notice the "remap" tags in the above code block, whose function is renaming the topics from the publisher and connect them to the subscriber. For example, the remap tag pair inside the circle detector node tag:

    <remap from="~image_compressed" to="usb_cam1/image_raw/compressed"/>

This line connect the "image_compressed" topic, which is defined in [code of the node](https://github.com/yxiao1996/RoboTop/blob/master/catkin_ws/src/10-objectdetection/circle_detector/src/circle_detector_node.py):

    class CircleDetectorNode():
        def __init__(self):
        ...
        self.sub_image = rospy.Subscriber("~image_compressed", CompressedImage, self.cbImage, queue_size=20)
        ...

To the "usb_cam1/image_raw/compressed" topic provided by the publisher usb_cam1 node. 