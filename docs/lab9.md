# LAB 9: Mapping 
The objective of this lab is to create a detailed layout (map) of  a fixed space, specifically the front room of the lab. This map is crucial for subsequent localization and navigation endeavors. To construct the map effectively, the procedure entails positioning the robot at designated points within the lab and instructing it to rotate on its axis while gathering Time-of-Flight (ToF) sensor data.

## Procedure 
The mapped area, encompassing the front room of the lab, is delimited by barricades outlining the designated map region, with strategically positioned obstacles. Four distinct marked points on the map indicate the locations for the robot to be stationed for capturing Time-of-Flight (TOF) readings.
With the origin at the center of the mapped area, the marked points are specified by the coordinates (-3, -2), (5, -3), (5, 3), and (0, 3). 

## Collating ToF Readings
To compile Time-of-Flight (ToF) readings at each of the four designated positions on the map, I programmed my robot to rotate about its axis approximately 15° every 1.5 seconds. This brief interval allows the ToF sensor to stabilize between readings. Each rotation cycle aims to collect an average of 25 readings per run. These readings are systematically gathered at every marked point to delineate a section of the room.

Once the ToF readings are gathered from each marked point, they're essentially puzzle pieces waiting to be put together. Using these readings, the map of the entire area can be pieced together. 

See below for the code for each cycle of collating ToF readings. This code runs in the main loop. 

```
if (mapping && yaw < 360.0){   // type(mapping) = bool. Toggled to begin taking ToF readings
  while (!distanceSensor2.checkForDataReady()) {
    delay(1);
  }
  mapping_tof_data = distanceSensor2.getDistance(); 
  car_spin(255);
  previousMillis = (int) millis(); 
  while ((int) millis() - previousMillis < 50){
     if (myICM.dataReady()){
        myICM.getAGMT();
        yaw = yaw + (-1 * myICM.gyrZ() * imu_dt); // imu_dt = 50 ms 
    }
  }
  stop_car(); 
```

## Mapping Regions  
### Coordinate: (-3, -2)  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/959db3ac-cc82-4adf-9c86-1f7dcf3a7261">  

### Coordinate: (5, -3)  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/ce387f8b-1617-48c4-b189-38e66b283b84">  

### Coordinate: (5, 3)  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/c5bbe826-9aaa-4cc4-8a4a-bbb269aa0634">  

### Coordinate: (0, 3  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/a9dac414-9243-4e6f-8824-df13e757c74e">  




### Other labs
* [LAB 10](lab10.md)

