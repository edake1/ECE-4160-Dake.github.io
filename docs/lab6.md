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
