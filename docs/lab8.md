# LAB 8: Stunts 
The purpose of this lab is to combine everything we have done so far to perform stunts with the robot. 

## Trial 1 - Toward the wall and back
For the stunt, I programmed my robot to dash towards a wall, perform a quick turn, and then backtrack along its original path. However, in the first trial, the orientation control malfunctioned as the robot approached the wall. Instead of executing a sharp turn, it veered off course, tracing a large arc and ultimately colliding with the wall.

This unexpected outcome could be attributed to the instructions in my code, which directed the robot to accelerate towards the wall and initiate the turn once it reached within 10 mm of the target setpoint, approximately 305 mm away (from the wall). Additionally, a low battery may have hindered the robot's ability to execute a sharp maneuver, resulting in subtle adjustment movements that inadvertently led to the collision. An isolation test conducted on the orientation control step revealed that the robot was executing subtle orientation adjustments instead of the intended sharp movement to position itself accurately.

```
previousMillis = (int) millis();
distanceSensor2.startRanging();
while (!distanceSensor2.checkForDataReady()) {
  delay(1);
  }
pid_distance1 = distanceSensor2.getDistance();

while ((int) millis() - previousMillis < 10000) {
  if (distanceSensor2.checkForDataReady()){
    pid_distance1 = distanceSensor2.getDistance();
  } 
  if (pid_distance1 > setpoint){
    car_forward(100); 
  } else {
    stop_car(); 
    delay(10); 
    car_spin(230); 
    delay(230); 
    stop_car(); 
    delay(10); 
    car_forward(200); 
    delay(2000); 
    stop_car(); 
    break;
  }
}
break; 
```


## Other Failed Trials 
[Wide orientation arc](https://youtube.com/shorts/R18-SjakcQw?feature=share)  

[Messy Orientation](https://youtube.com/shorts/F79bYBJAso0?feature=share)  

## Successful Attempts  

[Successful Attempt - 1](https://youtube.com/shorts/eXtnkrG7Kmo?feature=share)  

[Successful Attempt - 2](https://youtube.com/shorts/LarQ0VJX9j4?feature=share)  


## Final version 
[Final Stunt](https://youtube.com/shorts/YOKL7MZkM6U?feature=share)  


### Other labs
* [LAB 9](lab9.md)
