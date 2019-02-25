# Control API 

The below functions will help you to easily create highlevel path planning programs. This API is designed to be used with the ArduCopter flight stack. 
@ref control_functions

The following tutorial will show you how to make a simple program that allows you to send your drone to waypoints. 

First, we will clone the package Mission8_OutOfControls. 
```
git clone https://github.com/Texas-Aerial-Robotics/Mission8_OutOfControls.git
```

this package contains the file @ref control_functions.hpp . This file contains a bunch of useful functions for creating intelligent drone applications.

Once you have cloned Mission8_OutOfControls. Create a new file in called `control_tutorial.cpp` in `Mission8_OutOfControls/src`

First we will include our control functions
```
#include <control_functions.hpp>
```
This will allow us to use all of our control functions and structures 

Next add the main function and initialize ros
```
int main(int argc, char** argv)
{
	//initialize ros 
	ros::init(argc, argv, "offb_node");
	ros::NodeHandle controlnode;


	//Rest of code here


}
```

We will then add the function init_publisher_subscriber(). This function takes our ros node handle as an input and initializes subcribers that will collect the necissary information from our autopilot. add the following

```
//initialize control publisher/subscribers
init_publisher_subscriber(controlnode);
```
The control API contains the structure `control_api_waypoint` this structure contains the variables `x y z psi` which you can use to set locations and orientations of your drone. 

To make your drone fly in a square specify the following waypoints 
```
	//specify some waypoints 
	std::vector<control_api_waypoint> waypointList;
	control_api_waypoint nextWayPoint;
	nextWayPoint.x = 0;
	nextWayPoint.y = 0;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 0;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 5;
	nextWayPoint.y = 0;
	nextWayPoint.z = 3;
	nextWayPoint.psi = -90;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 5;
	nextWayPoint.y = 5;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 0;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 0;
	nextWayPoint.y = 5;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 90;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 0;
	nextWayPoint.y = 0;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 180;
	waypointList.push_back(nextWayPoint);
```
we will then add the following functions to handle preflight opperations 
```
	// wait for FCU connection
	wait4connect();

	//wait for used to switch to mode GUIDED
	wait4start();

	//create local reference frame 
	initialize_local_frame();
```
The function `wait4connect()` will loop until the node can communicate with the flight control unit (FCU). Once the connection with the FCU is established then we will use the function `wait4start()` to hold the program until the pilot executes the program by switching the FCU flight mode to GUIDED. This can be done from a ground control stattion (GCS) such as Mission Planner or QGroundControlv or from a switch on a radio controller. Finally, once the command to execcute the mission is sent you will use the function `initialize_local_frame()` to create your navigation frame. This function creates the local reference frame based on the starting location of the drone. 

Next we will request takeoff. using the function `takeoff(float takeOffHieght)`. Add
```
	//request takeoff
	takeoff(3);
```

Finally we will add our control loop
```
	//specify control loop rate. We recommend a low frequency to not over load the FCU with messages. Too many messages will cause the drone to be sluggish
	ros::Rate rate(2.0);
	int counter = 0;
	while(ros::ok())
	{
		ros::spinOnce();
		rate.sleep();
		if(check_waypoint_reached() == 1)
		{
			if (counter < waypointList.size())
			{
				set_destination(waypointList[counter].x,waypointList[counter].y,waypointList[counter].z, waypointList[counter].psi);
				counter++;	
			}else{
				//land after all waypoints are reached
				land();
			}	
		}	
		
	}
	return 0;
}
```
This loop will continually send the requested destination to the FCU via the `set_destination(x, y, z, psi)` function. The loop uses the function `check_waypoint_reached()` to determained when the drone has arrived at the requested destination. The function return 1 or 0. 1 denotes the drone has arrived. Each time the waypoint is reached the vector of waypoints is iterated to request the next waypoint via the variable `counter`. Finally, the drone will land via the land function `land()`


Your program should like like the following 
src/controlAPIExample.cpp 

```
#include <control_functions.hpp>
//include API 

int main(int argc, char** argv)
{
	//initialize ros 
	ros::init(argc, argv, "offb_node");
	ros::NodeHandle controlnode;
	
	//initialize control publisher/subscribers
	init_publisher_subscriber(controlnode);

	//specify some waypoints 
	std::vector<control_api_waypoint> waypointList;
	control_api_waypoint nextWayPoint;
	nextWayPoint.x = 0;
	nextWayPoint.y = 0;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 0;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 5;
	nextWayPoint.y = 0;
	nextWayPoint.z = 3;
	nextWayPoint.psi = -90;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 5;
	nextWayPoint.y = 5;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 0;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 0;
	nextWayPoint.y = 5;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 90;
	waypointList.push_back(nextWayPoint);
	nextWayPoint.x = 0;
	nextWayPoint.y = 0;
	nextWayPoint.z = 3;
	nextWayPoint.psi = 180;
	waypointList.push_back(nextWayPoint);

  	// wait for FCU connection
	wait4connect();

	//wait for used to switch to mode GUIDED
	wait4start();

	//create local reference frame 
	initialize_local_frame();

	//request takeoff
	takeoff(3);

	//specify control loop rate. We recommend a low frequency to not over load the FCU with messages. Too many messages will cause the drone to be sluggish
	ros::Rate rate(2.0);
	int counter = 0;
	while(ros::ok())
	{
		ros::spinOnce();
		rate.sleep();
		if(check_waypoint_reached() == 1)
		{
			if (counter < waypointList.size())
			{
				set_destination(waypointList[counter].x,waypointList[counter].y,waypointList[counter].z, waypointList[counter].psi);
				counter++;	
			}else{
				//land after all waypoints are reached
				land();
			}	
		}	
		
	}
	return 0;
}

```
run example code
---

```
roslaunch mission8_sim droneOnly.launch
# New Terminal
~/catkin_ws/src/Mission8_OutOfControls/scripts/startsim.sh
# New Terminal
roslaunch out_of_controls apm.launch
# New Terminal 
roslaunch out_of_controls contolAPIExample.launch
```
NOTE** you can tile gnome terminals by pressing `ctrl + shift + t`

You should now have a basic understanding of the functions available for controling your drone. You should be able to use these functions to help you make more complex navigation code.




