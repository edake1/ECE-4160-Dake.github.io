
# Localization on the real robot
In this lab, our objective is to implement localization using the Bayes filter on your physical robot. The primary goal is to grasp the distinctions between simulation and real-world systems. 

## Simulation 
### Final plot after running simulation  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/1b854423-b75e-4fc3-83e8-139c2d5d7ea0">  

### Implementing perform_observation_loop() function 

```
def perform_observation_loop(self, rot_vel=120):
  """Perform the observation loop behavior on the real robot, where the robot does  
  a 360 degree turn in place while collecting equidistant (in the angular space) sensor
  readings, with the first sensor reading taken at the robot's current heading. 
  The number of sensor readings depends on "observations_count"(=18) defined in world.yaml.
  
  Keyword arguments:
      rot_vel -- (Optional) Angular Velocity for loop (degrees/second)
                  Do not remove this parameter from the function definition, even if you don't use it.
  Returns:
      sensor_ranges   -- A column numpy array of the range values (meters)
      sensor_bearings -- A column numpy array of the bearings at which the sensor readings were taken (degrees)
                         The bearing values are not used in the Localization module, so you may return a empty numpy array
  """
  global localization_tof 
  sensor_ranges = np.zeros(18)[..., None] 
  sensor_bearings = np.zeros(18)[..., None]
  ble.send_command(CMD.SEND_MAPPING_MAPPING, "") 
  print("command to send data issued!!")
  while len(localization_tof) < 18: 
      asyncio.run(asyncio.sleep(4))
      print(localization_tof) 
      
  sensor_ranges = np.array(localization_tof)[np.newaxis].T
  sensor_bearings = np.array(localization_angles)[np.newaxis].T
  print("done collating data!") 
  return sensor_ranges, sensor_bearings
```

## Performing Localization at the four marked poses 
### (0ft, 3ft, 0°)  
The localization at the marked pose (0ft, 3ft, 0°) is shown below. You can observe that, although not perfect, the localization closely tracks the ground truth.  

<img width="700" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/e6ff5265-5eae-43c3-9ceb-ca75022b8104">  

The remaining waypoints were localized in a similar manner, showing a consistent trend as observed in the first localization. Despite minor offsets between the ground truth and the robot's belief, the waypoints were generally well localized. See the plots below for the remaining localized waypoints.

### (-3ft, -2ft, 0°)  
<img width="700" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/324a02c0-4c52-4570-9ded-c63eeef22001">  

### (5ft, 3ft, 0°)  
<img width="700" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/6b0818b2-2dc6-42f5-b43d-ab3410902136">  

### (5ft, -3ft, 0°)  
<img width="700" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/671c12d0-52cb-4576-90f8-83614973a099">  

In conclusion, the localization at all four marked poses was fairly consistent, with the robot's belief closely tracking the ground truth at each pose. However, despite multiple attempts, perfect localization (where the ground truth exactly matches the robot's belief) was not achieved. The plots above represent the best results from several localization trials at each pose.  

### Other labs
* [LAB 12](lab12.md)

