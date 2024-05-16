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

<!--
<div style="text-align: center;">
  <a href="https://www.youtube.com/watch?v=FUfWmMVqh1M" target="_blank">
  <img src="https://img.youtube.com/vi/FUfWmMVqh1M/0.jpg" alt="Watch the video" width="500" height="500">
</a>
  <p><em>Traversing Waypoints - Video</em></p>
</div>
-->
### Executing path traversal
My codebase suddenly exhibited unexpected behavior, causing all commands to hang midway through execution and resulting in Artemis disconnections, either during or immediately after command execution. This issue rendered my initial PID control function ineffective. Despite extensive troubleshooting efforts, including removing non-PID control code, rewriting the entire codebase within a given .ino file, meticulously reviewing each line of code, and eliminating unnecessary array instantiations that may be consuming memory, the problem persisted.

#### Revised path traversal plan 
After extensive tweaking, I repurposed my PID control function and streamlined my code by consolidating it into the main loop - I suspected that the code might be hanging during case command execution. The modified PID control function accepts a distance, and directs the robot to move forward or backward by the given distance (with very little correction so distances have to be precise). With this adjustment, I can now program the robot to navigate between waypoints by providing the distances needed to reach each destination. 

### Executing trajectory 
To execute the trajectory depicted in the image above (see [Grid with waypoints](#gridmap-with-marked-waypoints) section), a systematic approach is required. With nine distinct waypoints in the image, our trajectory execution algorithm is organized into nine precise steps, each aligned with a specific waypoint. At every step, the robot adheres to a customized sequence of commands designed to advance towards the subsequent waypoint. Key functions for forward and backward motion, derived from our PID control function, alongside orientation control, are indispensable components guiding this process.

[![Traversing Waypoints - Video](https://img.youtube.com/vi/FUfWmMVqh1M/0.jpg)](https://www.youtube.com/watch?v=FUfWmMVqh1M)

```
def move_forward(distance):
    command_string = f"|{distance}|3.10|0.01|0.02|"
    ble.send_command(CMD.PLANNING, command_string)

def make_turn(turn_angle, direction):
    move_direction = 1 # left
    if direction.lower() == "right": 
        move_direction = 2 
    command_string = f"|3.0|0.06|0.02|{turn_angle}|{move_direction}|" 
    ble.send_command(CMD.ORIENTATION_MOVEMENT, command_string)

def traverse():
    # waypoint 1: (-4, -3)
    move_forward(trajectory_distances[to_intermediate_wp])
    time.sleep(2)

    # intermediate waypoint (unlisted): (-2, -3) 
    make_turn(90, "left")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_2])
    time.sleep(4)

    # waypoint 2: (-2, -1) 
    make_turn(90, "right")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_3])
    time.sleep(4)

    # waypoint 3: (1, -1)
    make_turn(60, "right")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_4])
    time.sleep(4)

    # waypoint 4: (2, -3)
    make_turn(60, "left")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_5])
    time.sleep(4)

    # waypoint 5: (5, -3)
    make_turn(90, "left")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_6])
    time.sleep(4)

    # waypoint 6: (5, -2)
    move_forward(trajectory_distances[to_waypoint_7])
    time.sleep(4)

    # waypoint 7: (5, 3)
    make_turn(90, "left")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_8])
    time.sleep(4)

    # waypoint 8: (0, 3)
    make_turn(90, "left")
    time.sleep(4)
    move_forward(trajectory_distances[to_waypoint_9]) # ends at origin: (0, 0)
    time.sleep(4)
```

