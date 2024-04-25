# LAB 10: Grid Localization using Bayes Filter

The purpose of this lab is to implement grid localization using Bayes filter.

## Prelab 
### Execute Trajectory  

[Execute Trajectory](https://youtu.be/pl4ozlpD3XM)  

### Get Observation Data  

[Get Observation Data](https://youtu.be/4qUX-L_lzDs)  


## Bayes Filter  
Bayesian filtering in grid localization is a method for a robot to estimate its position within an environment. It divides the environment into a grid and updates its belief about its position using sensor measurements and control actions. By combining prior knowledge with new data, it provides a probabilistic estimate of the robot's location, handling uncertainty in both sensor readings and robot motion. For this lab segment, we'll implement key functions for the Bayes Filter:

* compute_control(cur_pose, prev_pose)
* odom_motion_model(cur_pose, prev_pose, u)
* prediction_step(cur_odom, prev_odom)
* sensor_model(obs)
* update_step()

### Implementation of Bayes Filter  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/b4b933b9-6624-4157-8e0e-019351d1e8b2">  


#### Compute_Control()  
The implementation of this function essentially boils down to computing the odometry model parameters.  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/cd8744f4-c238-476c-bb14-7fd4f5f7bdb7">  

```
def compute_control(cur_pose, prev_pose):
    """ Given the current and previous odometry poses, this function extracts
    the control information based on the odometry motion model.

    Args:
        cur_pose  ([Pose]): Current Pose
        prev_pose ([Pose]): Previous Pose 

    Returns:
        [delta_rot_1]: Rotation 1  (degrees)
        [delta_trans]: Translation (meters)
        [delta_rot_2]: Rotation 2  (degrees)
    """
    # compute delta_rot_1
    delta_rot_1 = np.degrees(np.arctan2(cur_pose[1] - prev_pose[1], cur_pose[0] - prev_pose[1])) - prev_pose[2]
    # normalize delta_rot_1
    delta_rot_1 = mapper.normalize_angle(delta_rot_1)
    
    # compute delta_rot_2
    delta_rot_2 = cur_pose[2] - prev_pose[2] - delta_rot_1
    # normalize delta_rot_2
    delta_rot_2 = mapper.normalize_angle(delta_rot_2)

    delta_trans = np.sqrt((cur_pose[1] - prev_pose[1]) ** 2 + (cur_pose[0] - prev_pose[1]) ** 2)
    

    return delta_rot_1, delta_trans, delta_rot_2


```


#### Odom_Motion_Model()  
The odometry motion model step assesses the likelihood of the robot transitioning from its prior position to its current one based on received control inputs. It factors in the uncertainties inherent in these movements, accounting for rotational and translational noise, which are modeled using Gaussian distributions in the implementation.

```
def odom_motion_model(cur_pose, prev_pose, u):
    """ Odometry Motion Model

    Args:
        cur_pose  ([Pose]): Current Pose
        prev_pose ([Pose]): Previous Pose
        (rot1, trans, rot2) (float, float, float): A tuple with control data in the format 
                                                   format (rot1, trans, rot2) with units (degrees, meters, degrees)


    Returns:
        prob [float]: Probability p(x'|x, u)
    """
    # utilize compute_control() to get transformation matrix components: (rot1, trans, rot2)
    rot1, trans, rot2 = compute_control(cur_pose, prev_pose)

    # We use loc.odom_rot_sigma and loc.odom_trans_sigma to model rotation and translation noise in odometry model
    prob = loc.gaussian(u[0], rot1, loc.odom_rot_sigma) * loc.gaussian(u[1], trans, loc.odom_trans_sigma) * loc.gaussian(u[2], rot2, loc.odom_rot_sigma) 

    return prob
```

#### Prediction_Step()  
For the prediction step, we are essentially computing the predicted probability distribution over possible robot poses in the grid map using the odometry model. At this step, we iterate over all possible poses in the grid map and calculate the likelihood of transitioning from the previous pose to each possible current pose using the odometry motion model (<em>odom_motion_model()</em> is used here). Then, the belief state is updated by integrating these transition probabilities with the prior beliefs for each pose.

See below for implementation of the <em>prediction_step()</em> function. 

```
def prediction_step(cur_odom, prev_odom):
    """ Prediction step of the Bayes Filter.
    Update the probabilities in loc.bel_bar based on loc.bel from the previous time step and the odometry motion model.

    Args:
        cur_odom  ([Pose]): Current Pose
        prev_odom ([Pose]): Previous Pose
    """
    # initialize values for max grid dimensions
    X = mapper.MAX_CELLS_X
    Y = mapper.MAX_CELLS_Y
    A = mapper.MAX_CELLS_A
    
    for x0 in range(X):
        for y0 in range(Y):
            for a0 in range(A):
                # update belief
                bel = loc.bel[x0][y0][a0]
                if bel >= 0.0001:
                    
                    #Loop through all possible current states
                    for x1 in range(mapper.MAX_CELLS_X):
                        for y1 in range(mapper.MAX_CELLS_Y):
                            for a1 in range(mapper.MAX_CELLS_A):
                                
                                #Prediction step
                                prob = odom_motion_model(mapper.from_map(x1, y1, a1), mapper.from_map(x0, y0, a0), u)
                                loc.bel_bar[x1][y1][a1] += (prob * bel)
    # normalize bel_bar 
    loc.bel_bar = loc.bel_bar / (np.sum(loc.bel_bar))
```


<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/67e362c2-cf51-4a3b-8b36-0bd68f4ae7a1">  


#### Sensor_Model()  
The sensor model step evaluates how likely it is to observe specific sensor measurements given a particular robot pose. This assessment is fundamental for updating the belief state in the Bayes filter, as it helps determine the contribution of sensor data to refining the robot's estimated pose.

```
def sensor_model(obs):
    """ This is the equivalent of p(z|x).


    Args:
        obs ([ndarray]): A 1D array consisting of the true observations for a specific robot pose in the map 

    Returns:
        [ndarray]: Returns a 1D array of size 18 (=loc.OBS_PER_CELL) with the likelihoods of each individual sensor measurement
    """
    prob_array = []
    for index in range(len(obs)): 
        prob = loc.gaussian(loc.obs_range_data[index][0], obs[index], loc.sensor_sigma)
        prob_array.append(prob)
    return prob_array

```

#### Update_Step()  
The update step refines the robot's belief about its pose by incorporating sensor measurements. It calculates the likelihood of observing the sensor data for each possible pose using the sensor model. Then, it updates the belief state by combining these likelihoods with the prior beliefs for each pose.  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/363456a8-7583-45d6-a8a0-83ebf4ea482d">  

```
def update_step():
    """ Update step of the Bayes Filter.
    Update the probabilities in loc.bel based on loc.bel_bar and the sensor model.
    """
    X = mapper.MAX_CELLS_X
    Y = mapper.MAX_CELLS_Y
    A = mapper.MAX_CELLS_A
    
    #Loop through all possible grid cell positions
    for x in range(X):
        for y in range(Y):
            for a in range(A):
                prob_z = np.prod(sensor_model(mapper.get_views(x, y, a)))
                loc.bel[x][y][a] = prob_z * loc.bel_bar[x][y][a]
    
    #Normalize probabilities to sum to 1
    loc.bel = loc.bel / np.sum(loc.bel)
```

### Running Bayes Filter in Simulator 
Using the helper functions implemented above, I ran my Bayes Filter implementation in the simulator. 

#### Simulator plotter color codes
* Red    - Odometry data
* Green  - Ground Truth
* Blue   - Belief of the robot. 

[Bayes Filter in simulator - Trial 1](https://youtu.be/siIbx5HlpbA)

[Bayes Filter in simulator - Trial 2](https://youtu.be/cQ0Zh27hDAo)    

[Bayes Filter in simulator - Trial 3](https://youtu.be/_nSl0o0BFZk)  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/28c4f55f-d0a5-403f-845b-a836114b3fe6"> 
<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/5958528b-6807-4894-9091-9df4054a6153">









