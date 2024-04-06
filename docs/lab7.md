
# LAB 7: Kalman Filter  
The objective of Lab 7 is to implement a Kalman Filter, which will help speed up the execution of the PID control behavior of halting about a few feet from a wall faster. 


<b>PARTS REQUIRED</b> 
- 1 x R/C stunt car
- 1 x SparkFun RedBoard Artemis Nano
- 2 x Dual motor drivers
- 1 x USB C-to-C or A-to-C cable
- 2 x 4m Time of Flight Sensors
- 1 x 9DOF IMU sensor 
- 1 x QWIIC Breakout board
- 2 x QWIIC connectors
- 1 x Li-Ion 3.7V 850mAh battery


## Kalman Filter Implementation
### Computing Parameters and Constructing Matrices
Before diving into the implementation details of the Kalman filter, it's crucial to compute the necessary parameters and construct the state space matrices. This initial setup lays the foundation for the subsequent filtering process.

```
# Finding parameters
ss_velocity  = 220
d = 1 / ss_velocity
rise_time = 2
m = (-d * rise_time) / np.log(0.1)
sampling_rate = 60
delta_t = 1 / sampling_rate

# Constructing A, B, and C matrices 
A = np.array([[0, 1], [0, (-d / m)]])
B = np.array([[0], [ 1 / m]])
C = np.array([[-1, 0]])

# Discretizing matrices A, and B
Ad = np.eye(2) + delta_t * A 
Bd = delta_t * B
```


### Filter Implementation 
The core of my Kalman filter implementation lies in the <em>kf(...)</em> function (see code snippet below), which performs the prediction and update steps of the Kalman filter algorithm. It takes the current state estimate, system covariance matrix, control input, and measurement as inputs and returns the updated state estimate and covariance matrix. 
For my implementation, I implemented a kalman_filter_output() function that utilizes the <em>kf(...)</em> function to process time-of-flight (TOF) data alongside corresponding timestamps and PWM values. It outputs the refined estimates of the system state based on the provided inputs - this data is what is plotted alongside the measured time-of-flight data to showcase the efficiency of the filter. 

```
# KALMAN FILTER IMPLEMENTATION
# Discretizing matrices A, and B
A, B = Ad, Bd

# Uncertainty of system model 
sigma_1, sigma_2 = 40, 50

# Uncertainty of sensor readings
sigma_3 = 1

sig_u = np.array([[sigma_1 ** 2,0],[0, sigma_2 ** 2]]) # from lab manual 
sig_z = np.array([[sigma_3 ** 2]]) # from lab manual 

def kf(mu, sigma, u, y):
    mu_p = A.dot(mu) + B.dot(u) 
    sigma_p = A.dot(sigma.dot(A.transpose())) + sig_u
    
    sigma_m = C.dot(sigma_p.dot(C.transpose())) + sig_z
    kkf_gain = sigma_p.dot(C.transpose().dot(np.linalg.inv(sigma_m)))

    y_m = y - C.dot(mu_p)
    mu = mu_p + kkf_gain.dot(y_m)    
    sigma=(np.eye(2) - kkf_gain.dot(C)).dot(sigma_p)

    return mu, sigma

def kalman_filter_output(tof_data, tof_time, pwm_value): 
    pwms = [pwm_value] * len(tof_time)
    output = []
    # initialize state of system 
    u = np.array([[tof_data[0]],[0]]) # use first time of flight data point as initial state
    sigma = sig_u
    for _, pwm, distance in zip(tof_time, pwms, tof_data):
        u, sigma = kf(u, sigma, pwm / pwm_value, distance)
        output.append(u[0][0])
    return output 

def error(arr1, arr2): 
    err = []
    err_2 = []
    for index, value in enumerate(arr1):
        err.append(value - arr2[index])
        err_2.append((value, arr2[index]))
    return err, err_2
```

### Kalman Filter Sanity Check 
Upon completing the Kalman filter implementation, I proceeded to validate its performance within the controlled environment of Jupyter Notebook before deploying it onto the robot. This validation process involved importing time-of-flight (TOF) data collected across four trials and conducting a thorough evaluation.

By plotting the measured TOF data alongside the filter's output against time, I conducted a rigorous assessment of its efficacy. These visualizations provided clear evidence of the filter's capability to accurately estimate the system's state even in the presence of noise and uncertainty.

In the pursuit of refining the model's accuracy, I experimented with various sigma values to define the uncertainty of the system model (sigma_1 and sigma_2) and the uncertainty of sensor readings (sigma_3). Through iterative experimentation, I identified the sigma values that best modeled the system's state, ultimately settling on the following values: sigma_1 = 40, sigma_2 = 50, and sigma_3 = 1. These values were found to yield optimal performance and effectively capture the dynamics of the system.

Below are the plots for each trial:  

#### Trial 1: sigma_1 = 40, sigma_2 = 50, sigma_3 = 1

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/2547f1e5-851b-4ccf-9406-d9e1839faeb1">  

#### Trial 2: sigma_1 = 40, sigma_2 = 50, sigma_3 = 1

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/16cc56df-a879-4d32-9331-bcafda3de294">  

#### Trial 3: sigma_1 = 40, sigma_2 = 50, sigma_3 = 1 

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/8a8ee827-8747-419b-8de1-2e655678df8a">  

#### Trial 4: sigma_1 = 40, sigma_2 = 50, sigma_3 = 1  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/6688fd5f-6479-4577-88ba-4bd15df61676">  


### Kalman Filter Implementation On Robot 
Following successful testing in simulated scenarios, the Kalman filter was integrated into the robot's control system. Known for its effectiveness in handling noisy data, the Kalman filter was implemented to improve state estimation in real-world conditions. Subsequent trials were conducted to evaluate its performance, providing valuable insights into its effectiveness in enhancing the robot's capabilities.  

<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/3b48bb33-8f1a-4bdc-86fa-c4b55ade855c">  
For the on-car Kalman Filter test, I activated both the Kalman Filter and PID control. The car was directed to move straight towards a wall with a setpoint of 305 mm (approximately 1 foot). As the car approached the wall, it gradually decelerated until reaching the setpoint, after which it exhibited minor deviations around the setpoint. Please refer to the accompanying plot and video for visual representation.  

[Kalman Filter On Robot](https://youtube.com/shorts/AKbp75emPto?feature=share)  
For the plot, it can be observed that there was a minor offset between the measured time-of-flight (TOF) sensor readings and the Kalman filter data. However, overall, the Kalman filter closely tracked the sensor readings and effectively smoothed out noise and fluctuations, resulting in a more consistent and accurate representation of the robot's position.

### Conclusion
In conclusion, the implementation and validation of the Kalman filter in this lab have proven to be instrumental in enhancing the accuracy and reliability of our system's measurements. Through meticulous experimentation and fine-tuning of parameters, we have successfully demonstrated the filter's ability to effectively estimate the system's state amidst noise and uncertainty.

By plotting the refined estimates alongside measured time-of-flight data, we have visually showcased the filter's efficiency in filtering out noise and providing robust state estimation. This not only validates the integrity of our Kalman filter implementation but also underscores its significance in real-world applications.







