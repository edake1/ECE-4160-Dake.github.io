# LAB 6: Orientation Control 
The purpose of this lab is to provide us with hands-on experience with orientation PID (Proportional-Integral-Derivative) using the IMU. 

<b>PARTS REQUIRED</b> 
- 1 x SparkFun RedBoard Artemis Nano
- 2 x Dual motor drivers
- 1 x USB C-to-C or A-to-C cable
- 2 x 4m Time of Flight Sensors
- 1 x 9DOF IMU sensor 
- 1 x QWIIC Breakout board
- 2 x QWIIC connectors
- 1 x Li-Ion 3.7V 850mAh battery

## Sending Data over BLE 
To send and receive IMU data for this lab, I used the <em>GET_IMU_DATA</em> and <em>GYROSCOPE</em> commands I implemented in the previous labs. The first command toggles a flag to allow the robot to start taking IMU data and the second command sends the pitch, roll, and yaw values computed from the gyroscope data. 
```
case GYROSCOPE: 
  Serial.println("Sending gyroscope pitch, roll, and yaw data ..."); 
  for (imu_index = 0; imu_index < IMU_DATA_SIZE; imu_index++){
    tx_estring_value.clear();
    tx_estring_value.append(imu_time[imu_index]);
    tx_estring_value.append(":");
    tx_estring_value.append(gyr_pitch[imu_index]);
    tx_estring_value.append(":");
    tx_estring_value.append(gyr_roll[imu_index]);
    tx_estring_value.append(":");
    tx_estring_value.append(gyr_yaw[imu_index]);
    tx_characteristic_string.writeValue(tx_estring_value.c_str());
    Serial.println(tx_estring_value.c_str());
          }
```
The code snippet above is the GYROSCOPE command used to send IMU data over BLE. 

## Orientation Control 
My orientation control loop essentially implemented logic to compute a PID value given a reference angle and the current orientation of the robot. Based on the PID value returned, the motors move in opposing directions to adjust the orientation of the robot till the desired orientation (reference angle) is reached. See the code snippet below for orientation loop logic. 

```
if (myICM.dataReady()){
  myICM.getAGMT();
  pid_imu_time[orient_index] = (int) millis(); 
  pid_imu_value = (int) imu_PID(reference_angle, orient_index, myICM.gyrZ(), 0.9, 0.0, 0.0);

  if (pid_imu_value >= 0){
    // move motor forward
    motor_L_speed = find_speed(pid_imu_value); 
    motor_R_speed = calibrated_pwm(motor_L_speed);

    analogWrite(6, 0);
    analogWrite(12, 0);          
    analogWrite(7, motor_L_speed); 
    analogWrite(0, motor_R_speed);
  } else {
    // move motor backward
    pid_imu_value = -pid_imu_value;
    motor_L_speed = find_speed(pid_imu_value); 
    motor_R_speed = calibrated_pwm(motor_L_speed);

    analogWrite(0, 0);
    analogWrite(7, 0);
    analogWrite(6, motor_L_speed); 
    analogWrite(12, motor_R_speed); 
  }
```

### PID value from IMU data
A function named imu_PID has been developed to calculate the PID value essential for motor adjustments to align the robot's orientation. This function accepts parameters such as the reference angle, gyroscope data, as well as the proportional (kp), integral (ki), and derivative (kd) constants. Upon processing, it yields the appropriate PID value.

When the discrepancy between the desired angle and the current orientation of the robot is positive, indicating a positive PID value, the function triggers analogWrite commands to manipulate the motors. This manipulation entails inducing opposite motions in the motors; for instance, the left motor advances while the right motor retreats. This counteractive motion generates a rotational effect about the Z-axis, facilitating the robot's alignment. Through repeated iterations, this corrective motion gradually guides the robot to align with the desired orientation. See below for the imu_PID function. 

```
float imu_PID(int ref_angle, int index, float gyrZ, float kp, float ki, float kd){
  // get yaw data and dt
  if (index < 1){
    imu_dt = 0.05; 
    yaw = 0.0; 
  } else {
    imu_dt = (pid_imu_time[index] - pid_imu_time[index - 1]) * 0.001;
    yaw = pid_imu_angles[index - 1] + -1 * (gyrZ * imu_dt);
  }

  // get error between desired angle and current angle
  imu_err = yaw - (float) ref_angle;

  // append error to error array
  pid_imu_error[index] = imu_err;

  // append yaw data 
  pid_imu_angles[index] = yaw; 

  // sum errors: integral 
  imu_integrator += imu_err * imu_dt;

  float derivative = gyrZ; 

  if (imu_integrator > 200){
    imu_integrator = 200; 
  }
  if (imu_integrator < -200){
    imu_integrator = -200; 
  }
 
  return (kp * imu_err) + (ki * imu_integrator) + (kd * derivative); 
}

```
### Trials 
#### P Controller: Kp = 5.0, Setpoint = 180°
I first implemented the proportional controller. By passing in a kp value of 5.0, my imu_PID() function returns the product of the error (desired orientation - current orientation) and the kp value. An <em>ORIENTATION_CONTROL</em> command was implemented in Arduino to oversee all orientation control logic. For the first trial, it is observed (see accompanying video below) that the robot jerks into position once the orientation control command is ran. When perturbations are introduced into the system, the robot tries to re-orient itself to minimize the error between the current orientation and the setpoint.  

[Trial 1](https://youtu.be/JP0k8hgsCTs)  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/fad8d2af-dade-4ae8-836c-46c93fa32133">

The graph above shows perturbations about the setpoint of 180° and the robot's attempts to re-orient itself to align with the setpoint. The peaks in the graph show the perturbations about the setpoint and the orange line serves as reference to show how the robot nicely snaps into place when disturbed. 
 
##### Trial 2
In the second trial, the robot tried to re-orient itself to assume the orientation of the setpoint but it's movement was in small increments. When you look at the graph below, you observe that the robot steadily approaches the setpoint albeit very slowly. From the graph, the adjustment from 0° to about 150° took about 3 seconds which means the robot moves about the Z-axis at a rate of approximately 50°/s.  

<img width="600" alt="Plot of time vs yaw" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/8388ab8e-e455-4bbc-8c82-14073b754fc7">

##### Trial 3 
Here, the robot moves steadily towards the setpoint of 180° but just like in the second trial, the movements are in small increments. In the trials so far, one set of wheels was visibly overpowering the other and thus responsible for most of the orientation adjustments. Getting both set of wheels to turn at comparable rates (may need to refine the wheel PWM calibration function) will speed up the rate at which the robot snaps into the desired direction.  

<img width="600" alt="Plot of time vs yaw" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/dd9c9ca0-67b7-456d-99e2-fa0961ba556f">  


#### Kp = 4.0, Setpoint = 180°
For a kp value of about 4.0, the robot adjusts itself to align with the setpoint but is usually a few degrees shy of the desired orientation. In all three trials (see plots below), the robot jerks quickly and almost aligns in the desired orientation and steadily oscillates about the setpoint the rest of the time.
##### Trial 1
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/04229dc1-f168-4105-98c8-f65fd882f61d">
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/96c3259b-6b5f-4149-90ab-395137e2909c">  

##### Trial 2 
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/9d867216-55e4-4102-b8c1-adaad5f09636">
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/958a6e6c-c9b0-4fb7-8138-6fd10665e013">  

##### Trial 3
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/7280464f-489a-4c69-b179-7d4753c3786b">
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/e64f76c8-9c14-4035-83ab-513bb9e8a247">


#### Kp = 3.0, Setpoint = 180°
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/72d12140-22c3-4dfc-9be0-5d11fc0a7d42">
<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/1c06051d-a70a-4467-b54e-89d35eddd5c0">

#### Sampling Rate
Throughout all the trials, I consistently gathered IMU measurements. The IMU provided data at a frequency of approximately 31 Hz, with measurements taken approximately every 32.3 milliseconds. During these trials, I logged every yaw data point recorded by the IMU for debugging purposes. However, this approach proved to be suboptimal due to the rapid filling of the array storing the yaw data, given the high sampling rate.

To address this issue, I made adjustments to the code. Now, only every fifth yaw data point collected during the robot's orientation operation is stored. This modification effectively reduces the strain on memory resources while still retaining essential data for analysis and debugging purposes.


#### Kp Discussion 
I successfully implemented the proportional (P) controller for orientation control, achieving satisfactory performance with a Kp value of approximately 4.0. Upon experimentation, I observed that for Kp values exceeding 5 and falling below 3, the robot took significantly longer to reach a stable equilibrium point. Moreover, even when it approached stability, it consistently fell a few degrees short of the desired orientation.

In contrast, with a Kp value around 4.0, although the robot still slightly undershoots the target orientation by a few degrees, it reaches a stable equilibrium point in considerably less time—typically less than a second. This demonstrates a favorable balance between speed of convergence and precision in orientation control achieved with an optimal Kp value.


### Other labs
* [LAB 7](lab7.md)
* [LAB 8](lab8.md)
* [LAB 9](lab9.md)
* [LAB 10](lab10.md)
* [LAB 11](lab11.md)
* [LAB 12](lab12.md)

