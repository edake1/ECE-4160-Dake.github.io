
# LAB 5: Linear PID control and Linear interpolation
This purpose of this lab is to help us get familiar with PID control. In this document, I discuss my implementation of a PID controller for my robot. 

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




