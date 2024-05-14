
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


### Noise in accelerometer frequency spectrum 
With the IMU laid still on the table with no externally induced vibrations, readings for the accelerometer pitch and roll were taken and the results are shown in the image below. Since there was no externally induced high frequency noise, we do not expect the frequency spectrum to show unusually sharp peaks at various frequencies, and this is corroborated by the fourier transform plot of the accelerometer roll and pitch data in the right image (see below). 

<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/7e4af40f-6fa5-44e6-9856-76021784f9d2"> <img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/bd5c60ea-7b5f-4aea-b039-b3f691093e53">

### Low Pass Filter  
Using the code below, a low pass filter was implemented for the accelerometer. 
```
  ...
  acc_pitch_lpf[imu_index] = alpha * acc_pitch[imu_index] + (1 - alpha) * acc_pitch_lpf[imu_index - 1]; 
  acc_pitch_lpf[imu_index - 1] = acc_pitch_lpf[imu_index]; 
  acc_roll_lpf[imu_index] = alpha * acc_roll[imu_index] + (1 - alpha) * acc_roll_lpf[imu_index - 1]; 
  acc_roll_lpf[imu_index - 1] = acc_roll_lpf[imu_index];
  ...
```
Even without inducing some vibrational noise into the system, after passing the accelerometer data through the complimentary low pass filter, you can observe that the low pass filtered plot (see image below) appears smoother than the original roll and pitch data. This goes to show that some noise was caught by the filter.   

<img width="1000" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/d7caa199-ceb1-48e3-acb0-8c3841451e00">

With little perturbations in the system, the accelerometer and gyroscope data also show oscillations about a mean value with no unusually sharp peaks.  

<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/d2839225-6771-4cff-a3cc-a97731f94fcc">

#### Inducing vibrational noise into the system  
On inducing vibrational noise into the system by tapping the table, I collated accelerometer data and plot the fourier transform of the roll and pitch. Even with the induced vibrational noise in the system, the fourier spectrum shows almost no sharp peaks except for the slight peak at about 3.5Hz and 4.8Hz (see image below). An internal filter would explain why even with the externally induced noise the frequency spectrum does not show much high frequency noise in the system.  

<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/30ba5d21-53dd-4732-9d53-4eb8bba282a4">  

On the specifications sheet of the IMU, we see the line "On-Chip 16-bit ADCs and Programmable Filters" under the features section which supports our observation. The good resolution offered by the 16-bit ADC coupled with the presence of the programmable filter would explain our observation above. Due to the in-built noise rejection ability of the IMU, an extra noise filter may not be necessary but I implemented one and the results were noticeable which was a little unexpected given the fact that the chip had already filtered some of the noise. I used a moderate alpha value of 0.4 for my low pass filter.  

<img width="1319" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/432f4483-c712-4981-9949-e35a3afa4f34">  

## Gyroscope  
Referencing the gyroscope equations from lectures and using the code below, I computed the gyroscope pitch and roll. 
```
  ...
  gyr_pitch[imu_index] = gyr_pitch[imu_index - 1] + myICM.gyrY() * dt; 
  gyr_roll[imu_index] = gyr_roll[imu_index - 1] + myICM.gyrX() * dt; 
  gyr_yaw[imu_index] = gyr_yaw[imu_index - 1] + myICM.gyrZ() * dt;
  ...
```
Even with externally induced noise, the gyroscope data seemed fairly stable and the fourier spectrum plot is very similar to that of the accelerometer pitch and roll. Using the code below, we implemented the complimentary filter for the gyroscope and the plot for the filtered gyroscope data is shown below. I chose an alpha value of about 0.4 for my filter. 

```
  ...
  T = imu_time[imu_index] - imu_time[imu_index - 1]; 
  dt = T * 0.001; // get dt in seconds 
  alpha = 0.4;
  gyr_pitch_cf[imu_index] = (gyr_pitch_cf[imu_index] + myICM.gyrY() * dt) * (1 - alpha) + alpha * acc_roll[imu_index];
  gyr_roll_cf[imu_index] = (gyr_pitch_cf[imu_index] + myICM.gyrY() * dt) * (1 - alpha) + alpha * acc_roll[imu_index];
...
```

<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/147cfffe-2dd5-4548-9ffc-25b03bff09f1">  <img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/74d06f9f-5d1d-4912-8e18-2d62738a7b43">  


## Data Sampling 
### Speed of main loop 
In the main loop, I focused on using if-expressions to determine whether to execute tasks, such as data collection, to avoid additional loops. This approach significantly enhanced the main loop's speed/efficiency and increased the data sampling rate from the IMU, especially when avoiding other loops and serial print statements.  

<img width="1000" alt="sampling-rates" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/2ca3b159-1e8c-40d6-86ba-2d31c9676ef7">  

During each iteration of the main loop, I aggregated timestamps corresponding to when the IMU collected data and transmitted them via BLE. Subsequently, I implemented a handler to analyze these timestamps and compute the time differences between each timestamp and its preceding one. This analysis enabled me to estimate the IMU's sampling rate. Notably, the main loop executed approximately once every <em>0.03 milliseconds</em>, while the IMU sampled values roughly every <em>18.3 milliseconds</em>. Based on this data, it appears that the main loop on the Artemis runs approximately <em>600 times</em> faster than the IMU's rate of producing new values.


### Data storage and transfer protocol
To manage data collection and transmission over Bluetooth Low Energy (BLE) in my project, I first implemented flags in the code that signal when to collect data. The collected data is temporarily stored in an array, and a separate command triggers its transfer over BLE to my computer. This method of storing before transmitting was chosen to avoid potential latency that could occur if data were sent immediately upon collection, which might slow down the robot's movement and responsiveness to commands.

Additionally, I decided to send different types of data in separate batches, rather than using one large array for everything. For example, gyroscope data, complete with timestamps, are stored and transmitted in a distinct array from accelerometer data. This approach is tailored to the BLE's data transfer limit of about 150 characters.

And, for transmission efficiency, data is sent over BLE as strings, rather than in their original data types (such as int or float). This strategy is more memory-efficient and helps maintain the system’s performance within the constraints of BLE communication.

### Five seconds worth of data
I implemented a command that collated five seconds worth of IMU data and transmitted it over BLE to the computer. The transmitted data includes the current timestamp, accelerometer pitch, and accelerometer roll values. After approximately ten transmissions, I achieved an average of 314 strings transmitted each time.  
<img width="1000" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/346596cc-1e70-4753-9f52-2f490206196d">


## STUNT 
After testing my RC car, I was impressed by its performance. It responded quickly and accurately, adeptly performing flips, quick turns, and spins with specific key commands. Notably, the car's ability to abruptly stop at full speed and execute a flip was particularly striking, and I'm excited to replicate this maneuver. Details of the car's stunts are provided below.

[Car Stunt](https://www.youtube.com/shorts/sbtbLbjgg3Y)  



### Other labs
* [LAB 3](lab3.md)
* [LAB 4](lab4.md)
* [LAB 5](lab5.md)
* [LAB 6](lab6.md)
* [LAB 7](lab7.md)
* [LAB 8](lab8.md)
* [LAB 9](lab9.md)
* [LAB 10](lab10.md)
* [LAB 11](lab11.md)
* [LAB 12](lab12.md)






