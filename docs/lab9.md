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


#### Generating Global Map 
From the plotted regions depicted above, it's apparent that each individual map appears to be shifted by approximately 45°. This discrepancy was unforeseen, given that precautions were taken during data collection to prevent such offsets. Nevertheless, we can realign the data points and assemble the individual maps to reconstruct the global map.

##### Transforming datapoints to global map 
Each of the aggregated data points resides within its respective local space, necessitating a transformation process to integrate them into the global map. Below, you'll find the function employed for precisely this purpose.  

```
def transform_data4(dict_data):
  new_all_data = []
  for key, value in dict_data.items(): # key = (x, y), value = array of array datapoints
    x, y = key # center coordinates 
    result = []
    for angle, tof in value:
      # Transformation matrix
      TMatrix = np.array([[np.cos(radians(angle)), -np.sin(radians(angle)), x], 
                          [np.sin(radians(angle)), np.cos(radians(angle)), y], 
                          [0, 0, 1]])
      # matrix to transform 
      val = np.array([[tof], 
                      [0],
                      [0]])
      data = np.matmul(TMatrix, val)
      result.append(data)
    new_all_data.append(data)
return new_all_data

```

Upon transforming all the points, the global map was derived. However, even after the initial attempt at transformation, the map remained misaligned, as illustrated below.

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/849c02b5-0dae-489f-ac35-dd9a0cf786ba">  

Given the inaccuracies present in the map, it became imperative to diagnose the underlying issue, meticulously reassemble the points, and subsequently derive a significantly more precise map.

Upon observation, I attempted to align all the points by rotating them. However, it became apparent that the issue persisted even after rotation, suggesting that the problem might stem from erroneous central coordinates used for plotting.  

<img width="500" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/4f2a5250-3775-4635-a5c0-acbace1f72cc">  

### Other labs
* [LAB 10](lab10.md)

