# Lab 12: Path Planning and Execution

The objective of this lab is to have the robot navigate through a set of waypoints in that environment as quickly and accurately as possible.


## Traversing Waypoints
A total of 9 distinct waypoints were scattered across the entire map, each with its unique coordinates:
1. (-4, -3)    <--start
2. (-2, -1)
3. (1, -1)
4. (2, -3)
5. (5, -3)
6. (5, -2)
7. (5, 3)
8. (0, 3)
9. (0, 0)      <--end

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/69a18795-ff30-4d3d-91e5-cb7b311374f6">

### Initial (ideal) plan
The initial plan involved utilizing PID control functionality in conjunction with orientation control commands to facilitate the robot's navigation between predefined waypoints. These waypoints, located within a gridded area outside the lab, were demarcated with defined coordinates (see waypoint coordinates above). Prior to implementation, I conducted precise geometric calculations to determine inter-waypoint distances (made easier by the fact that each tile in the grid was about one foot in width and length). The intended approach was to initiate the robot's movement from the first waypoint, utilizing PID control to navigate towards subsequent waypoints while employing orientation control for precise turns. Incorporating PID control enabled the robot to perform necessary backup and repositioning maneuvers in cases of overshooting or undershooting the intended distance.  

[Traversing Waypoints - Video](https://youtu.be/FUfWmMVqh1M)
