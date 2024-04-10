# LAB 8: Stunts 
The purpose of this lab is to combine everything we have done so far to perform stunts with the robot. 

## Trial 1 - Toward the wall and back
For the stunt, I programmed my robot to dash towards a wall, perform a quick turn, and then backtrack along its original path. However, in the first trial, the orientation control malfunctioned as the robot approached the wall. Instead of executing a sharp turn, it veered off course, tracing a large arc and ultimately colliding with the wall.

This unexpected outcome could be attributed to the instructions in my code, which directed the robot to accelerate towards the wall and initiate the turn once it reached within 10 mm of the target setpoint, approximately 305 mm away (from the wall). Additionally, a low battery may have hindered the robot's ability to execute a sharp maneuver, resulting in subtle adjustment movements that inadvertently led to the collision. An isolation test conducted on the orientation control step revealed that the robot was executing subtle orientation adjustments instead of the intended sharp movement to position itself accurately.


[Stunts](https://youtube.com/shorts/R18-SjakcQw?feature=share)
