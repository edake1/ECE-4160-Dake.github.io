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

### Grid(map) with marked waypoints
See the photo below for a diagram of the map showing the waypoints and the ideal trajectory for inter-waypoint traversal.  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/69a18795-ff30-4d3d-91e5-cb7b311374f6">  

### Initial (ideal) plan
The initial plan involved utilizing PID control functionality in conjunction with orientation control commands to facilitate the robot's navigation between predefined waypoints. These waypoints, located within a gridded area outside the lab, were demarcated with defined coordinates (see waypoint coordinates above). Prior to implementation, I conducted precise geometric calculations to determine inter-waypoint distances (made easier by the fact that each tile in the grid was about one foot in width and length). The intended approach was to initiate the robot's movement from the first waypoint, utilizing PID control to navigate towards subsequent waypoints while employing orientation control for precise turns. Incorporating PID control enabled the robot to perform necessary backup and repositioning maneuvers in cases of overshooting or undershooting the intended distance.  

[Traversing Waypoints - Video](https://youtu.be/FUfWmMVqh1M)


[![Watch the video](https://img.youtube.com/vi/FUfWmMVqh1M/0.jpg)](https://www.youtube.com/watch?v=FUfWmMVqh1M)


<iframe width="560" height="315" src="https://www.youtube.com/embed/FUfWmMVqh1M" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Executing path traversal
My codebase suddenly exhibited unexpected behavior, causing all commands to hang midway through execution and resulting in Artemis disconnections, either during or immediately after command execution. This issue rendered my initial PID control function ineffective. Despite extensive troubleshooting efforts, including removing non-PID control code, rewriting the entire codebase within a given .ino file, meticulously reviewing each line of code, and eliminating unnecessary array instantiations that may be consuming memory, the problem persisted.

#### Revised path traversal plan 
After extensive tweaking, I repurposed my PID control function and streamlined my code by consolidating it into the main loop - I suspected that the code might be hanging during case command execution. The modified PID control function accepts a distance, and directs the robot to move forward or backward by the given distance (with very little correction so distances have to be precise). With this adjustment, I can now program the robot to navigate between waypoints by providing the distances needed to reach each destination. 



