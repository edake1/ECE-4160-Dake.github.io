
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
I added code to blink the in-built LED on start up to ascertain that the sensor is about to take values. 

### AD0_VAL
In the default example file, the AD0_VAL value was set to 1 but when the code was run with this value, we get a <em>data underflow</em>. I quickly changed the AD0_VAL to 1 and it ressolved the issue. The AD0_VAL is concerned with configuring the state of the AD0 pin (pin used to set the address of the sensor) of the sensor. Changing the AD0_VAL to 1 set the pin high in this case.  
<img width="300" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/1c16fea4-c12d-4c58-8d62-4bf82b2d5493">  

### Testing the IMU
I tested the IMU by turning it in different directions and slowly accelerating it. From this test, we observe that when the IMU is positioned exactly in any of the three axes, the accelerometer reading for that axis has a greater magnitude (~1000 mg) compared to the others. When the sensor is positioned in any other orientation, the values recorded on the axes is adjusted accordingly with axis recording a greater value indicating that the force of gravity is stronger in that direction. 
The gyroscope also recorded data on the orientation of the sensor such that when I oriented the sensor horizontally in a given axis, the reading on that axis is about zero. A vertical orientation in any of the axis also records a magnitude of about 90 degrees in that direction. 


## Accelerometer  
I extracted the data recorded by the accelerometer and using the code below, I converted it to pitch and roll.
```
...
float a_x = myICM.accX(); 
float a_y = myICM.accY();
float a_z = myICM.accZ();

float pitch = atan2(a_x, a_z) * 180.0 / M_PI; 
float roll = atan2(a_y, a_z) * 180.0 / M_PI;
...
```
Using the edge of the lab table as reference, orienting the sensor to get 90 degrees pitch and roll resulted in the output below:  
<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/b18b5a0d-d5e2-4df9-878c-b83e2a2aea04">  

<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/13693d13-a2f3-4da1-b3d2-9041f1080a14">  

### Noise in accelerometer frequency spectrum 
Even without inducing some vibrational noise into the system, after passing the accelerometer data through the complimentary low pass filter, you can observe that the low pass filtered plot (see image below) appears smoother than the original roll and pitch data. This goes to show that some noise was caught by the filter. 
<img width="1291" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/4f18c674-753b-446f-88f9-a86a7b1052eb">  

Due to the little noise in the system, as expected, the fourier spectrum of the accelerometer roll and pitch data does not show any unusually sharp peaks or high frequency noise in the system.  
<img width="761" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/f62e9759-714b-42f9-ac90-fbbd3b754345">  






