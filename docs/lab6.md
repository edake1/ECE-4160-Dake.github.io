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

### Orientation Control 
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

#### PID value from IMU data  

```
float imu_PID(int ref_angle, int index, float gyrZ, float kp, float ki, float kd){
  // get yaw data and dt
  if (index < 1){
    imu_dt = 0.05; 
    yaw = 0.0; 
  } else {
    imu_dt = (pid_imu_time[index] - pid_imu_time[index - 1]) * 0.001;
    yaw = pid_imu_angles[index - 1] + gyrZ * imu_dt;
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

### Tuning 

