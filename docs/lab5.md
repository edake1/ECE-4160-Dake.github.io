
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
