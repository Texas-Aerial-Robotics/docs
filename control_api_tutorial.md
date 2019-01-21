# Control API 

The below functions will help you to easily create highlevel path planning programs. This API is designed to be used with the ArduCopter flight stack. 
@ref control_functions

The below code is a simple example that executes a square flight pattern. 
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