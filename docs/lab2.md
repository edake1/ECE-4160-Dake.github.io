
# LAB 2: IMU

The purpose of this lab is to equip the robot the Inertial Measurement Unit (IMU) sensor which provides data on the movement, orientation, and position of the robot. 

<b>PARTS REQUIRED</b> 
- 1 x SparkFun RedBoard Artemis Nano
- 1 x USB C-to-C cable
- 1 x 9DOF IMU Sensor 
- 1 x Force1 RC Car 
- 1 x Li-Ion 3.7V 850mAh battery

## IMU Setup 
Connecting the IMU sensor to the Artemis board was straightforward as it involved simply connecting the IMU to the board via a single QWIIC connector. After the IMU was connected, I tested it by running the <em>Example1_Basics.ino</em> file in the SparkFun IMU library. The output shows the IMU actively recording accelerometer, gyroscope, and magnetometer values.  
<img width="800" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/49150357-e8a7-4e1f-848f-5e17649595ad">

### AD0_VAL: In the default example file, the AD0_VAL value was set to 1 but when the code was run with this value, we get a <em>data underflow</em>. I quickly changed the AD0_VAL to 1 and it ressolved the issue. 
<img width="300" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/1c16fea4-c12d-4c58-8d62-4bf82b2d5493">





```

### Sensor placement 
To ensure accurate obstacle detection, sensor placement is critical. I positioned one sensor at the front and another at the back of the car. This setup enables the robot to detect obstacles from both its front and rear positions, providing comprehensive situational awareness. Alternatively, installing both sensors at the front would have limited the robot's detection capabilities to only the front, potentially leaving blind spots at the rear.

### Time of Flight Sensor Modes 
The TOF sensor has some flexibility in the mode for taking measurements based on the range needed. You can either set the distance ranging mode to short or long via the <em>setDistanceModeShort()</em> and <em>setDistanceModeLong() methods. I decided to go with the short distance mode because I figured if the robot is going to be performing stunts, I want it to be maximally sensitive and accurate. The long distance mode sacrifices sensitivity to ambient noise for greater range so the short distance mode was a better option in this case. 

###Short Distance Mode Statistics
#### Range: 10mm - 1686 mm 
To determine the range, I aimed my TOF sensors at a nearby wall and steadily moved backward until they were out of range. I recorded the distances measured by the TOF sensors at the instant they were out of range multiple times and averaged the results to establish the farthest range which was about <em>1686 mm</em>. 


#### Repeatability
Here, I took 10 separate measurements at distances of 1500mm and 500mm to ascertain how repeatable the measurements are. In both measurements, the measured data was around the same value but with deviations of up to 4mm. In the 1500mm case, most of the measured distances deviated from the expected value by only 1mm, and in the 500mm case, it was about 1 - 2mm. 
<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/2c02bb69-0a0a-4971-a952-d9cf5a1e2cde">  <img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/af0eee43-5fb8-4fdc-821e-a027d997b3f9">

#### Accuracy: 1.8mm 
Using the same data from the repeatability measurements, I averaged the deviations from the measurements and calculated the accuracy to be around 4mm of the expected value. 










