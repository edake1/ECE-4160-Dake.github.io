
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
Computing parameters 
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


Filter Implementation 
```
# KALMAN FILTER IMPLEMENTATION
A, B = Ad, Bd 
sigma_1, sigma_2, sigma_3 = 50, 50, 2
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
    u = np.array([[tof_data[0]],[0]])
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
Trial 1
<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/2547f1e5-851b-4ccf-9406-d9e1839faeb1">  
Trial 2  
<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/16cc56df-a879-4d32-9331-bcafda3de294">  
Trial 3  
<img width="600" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/8a8ee827-8747-419b-8de1-2e655678df8a">  






