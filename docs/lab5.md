
# LAB 5: Linear PID control and Linear interpolation
The purpose of this lab is to provide us with hands-on experience and practical understanding of PID (Proportional-Integral-Derivative) control. Throughout this document, I delve into the intricacies of my PID controller implementation tailored specifically for my robot. 

<b>PARTS REQUIRED</b> 
- 1 x SparkFun RedBoard Artemis Nano
- 2 x Dual motor drivers
- 1 x USB C-to-C or A-to-C cable
- 2 x 4m Time of Flight Sensors
- 1 x QWIIC Breakout board
- 2 x QWIIC connectors
- 1 x Li-Ion 3.7V 850mAh battery

## Storing Debugging Data 
To be able to ascertain the behavior of my robot when attempting to execute commands and debug unexpected behavior, there was the need to store data about the state of the robot at any point in time when it is executing commands. I wrote commands to store time of flight sensor data, the current time, and the PID values being used by the robot in arrays while the robot is executing an action (e.g. moving towards the wall at full speed). At any point in time during the robot's motion, I can send a command over BLE to the Artemis to send over the debugging data to the computer for analysis. I implememnted a handler on the computer side to parse this debugging data for debugging. 
The code snippet below shows how I send debugging data stored in arrays over BLE to the computer for analysis.  

```
case PID_DATA:
  for (pid_index = 0; pid_index < PID_DATA_SIZE; pid_index++){
    tx_estring_value.clear();
    tx_estring_value.append(pid_time[pid_index]);
    tx_estring_value.append(":");
    tx_estring_value.append(pid_tof1[pid_index]);
    tx_estring_value.append(":");
    tx_estring_value.append(pid_value);
    tx_estring_value.append(":");
    tx_estring_value.append(pid_error[pid_index]);
    tx_characteristic_string.writeValue(tx_estring_value.c_str());
    Serial.println(tx_estring_value.c_str());
    }
  break;
...
```
## PID Controller 
### Proportional Controller
The goal was to program the robot to approach a nearby wall and stop at a set distance from it. For testing purposes, the chosen setpoint was 1 foot or approximately 305 mm.

To achieve this, I implemented a Proportional (P) controller, which dynamically adjusts the robot's speed based on its current position relative to the wall. This involved continuously calculating the error between the setpoint and the robot's position and modulating the motor speed accordingly.
In essence, the robot moves faster when far from the wall and gradually slows down as it approaches the setpoint, ensuring precise positioning.
See the code snippet below for implementation details.  

```
...
if (motor_run && pid_index < PID_DATA_SIZE){
  // get current distance 
  pid_distance1 = distanceSensor1.getDistance();
  // get current time in milliseconds and append to array
  pid_time[pid_index] = (int) millis(); 
  // append distance reading to array
  pid_tof1[pid_index] = pid_distance1;
  // find pid value 
  pid_value = PID(pid_index, setpoint, kp, ki, kd, pid_distance1);
  pid_values[pid_index] = pid_value; 

  if (pid_value >= 0){
    // move forward
    motor_L_speed = find_speed(pid_value); 
    motor_R_speed = calibrated_pwm(motor_L_speed); 
    analogWrite(6, motor_L_speed); 
    analogWrite(7, 0);
    analogWrite(0, motor_R_speed);
    analogWrite(12, 0);
  } else {
    // move backward
    pid_value = -pid_value;
    motor_L_speed = find_speed(pid_value); 
    motor_R_speed = calibrated_pwm(motor_L_speed);
    analogWrite(6, 0);
    analogWrite(0, 0);
    analogWrite(7, motor_L_speed); 
    analogWrite(12, motor_R_speed); 
  }
... ... 
```

There is a some disparity in the PWM values at which my motors spin at relatively similar speeds so I implemented a calibration factor that ensures that for a given PID value, the motors spin at similar speeds to avoid unnecessary turns or rotations during motion. The function <em>find_speed()</em> in the code snippet above essentially takes a PID value and ensures it is within the range of PWM values, (0, 255) and also adjusts the speed if the PID value is within range but lower than the lower PWM limit of each of the motors. Meanwhile, the function <em>calibrated_pwm()</em> in the code snippet above applies the calibration factor to ensure that the other motor spins at a comparable speed.

I implememted a <em>PID()</em> function which is the core of the Proportional controller. It takes a certain setpoint, the current position of the robot, and a kp value and returns an adjusted value which is <em>proportional</em> to the error between the current position of the robot and the setpoint. 

```
float PID(int index, int set_point, float kp, float ki, float kd, int distance){
// get error 
pid_err = set_point - distance; 
// append error to error array
pid_error[index] = pid_err;

// sum errors - integral 
integrator += pid_err * pid_dt;
if (integrator > max_integral_value) {
  integrator = max_integral
}
if (integrator < -max_integral_value){
  integrator = -max_integral_value
}
// find derivative 
if (index >= 1){
  derivative = -(pid_error[index] - pid_error[index - 1]) / pid_dt; 
} else {
  derivative = 0.0; 
}
return kp * pid_err + kd * derivative + ki * integrator; 
}
```
The above implementation does not factor in the I and D controllers yet so it just outputs a value proportional to the current error in position relative to the setpoint. 

#### Tuning 
For my first test, I just decided to test with a high Kp value just to see the behavior of the robot. And, at a value of 1, the robot sped forward quickly and just bumped head on into the wall before attempting to back up a little. After this observation, I proceeded to test with very small Kp values and adjust accordingly. 
##### Kp = 0.01 
<img width="600" alt="Plot of time vs distance" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/14c8e48b-a279-43e0-a733-11af7b05bcf4">  

The plot above is for two iterations of the test. You can observe from the plot that the robot first overshot and was about 125 mm away from the wall, then it backed up to about 360 mm then proceeded to steadily position itself at the setpoint in small oscillations. The second peak at about <em>1500 mm</em> was as a result of changing the position of the robot for another run. Again, a similar behavior is observed where the robot overshoots and backs up to the setpoint.

##### Trials (Kp = 0.02)  
[Trial 1](https://youtube.com/shorts/-_K9_mgNgjU?feature=share)  

TOF sensor returns data every 63.8 ms or at 15.7 Hz 

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/91ce6a13-6987-4b95-98c1-884fa67bbe9e">   

[Trial 2](https://youtube.com/shorts/j6J2BNOmuCI?feature=share)  

TOF sensor returns data every 64.6 ms or at 15.5 Hz  

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/f49850b6-6696-4723-83b5-4e1b094b39bf">    
  
[Trial 3](https://youtube.com/shorts/YJJzpQZE2X0?feature=share)  

TOF sensor returns data every 90.9 ms or at 11 Hz

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/ba8b9ddc-03f5-471f-9b2e-822db80be239">  

Over the course of the three trials, the robot was moving at an average velocity of -0.12 m/s or -0.12 mm/ms.  

The results observed at a Kp value of 0.02 were fairly good but the amplitude of oscillations around the setpoint were a little large so I decided to test with other kp values to arrive at a better kp value. See below for plots of other trials. 

##### Kp = 0.01 

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/e0429307-1080-4faf-9577-ebfe29788cba">  

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/37041fee-cedf-423e-8b1d-1960e103c326">  

##### Kp = 0.012  

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/ea407ad6-9ea9-477f-8c7b-4ca405dc13f5">

##### Kp = 0.017  

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/f046dc9f-8382-45cb-86c0-a1fd35ce50a1">  

<img width="722" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/ca293111-0d7b-4940-b13b-be8eefa3bd86">








