# LAB 4: Motors and Open Loop Control
This purpose of this lab is to equip the robot with time of flight sensors to enable it judge distances accurately in order to avoid running into obstacles and perform stunts. 

<b>PARTS REQUIRED</b> 
- 1 x SparkFun RedBoard Artemis Nano
- 2 x Dual motor drivers
- 1 x USB C-to-C or A-to-C cable
- 2 x 4m Time of Flight Sensors
- 1 x QWIIC Breakout board
- 2 x QWIIC connectors
- 1 x Li-Ion 3.7V 850mAh battery

## The Motor Drivers 
<img width="300" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/3429ca6d-ef52-4d19-8248-94e7ce5f4eea">  

### Wiring diagram  
As can be seen from the wiring diagram below, instead of driving both motors with the two inputs and outputs, we decided to parallel-couple the inputs and outputs on each motor driver and power them separately. By doing this, we are able to deliver twice the average current without running the risk of overheating the chip thus enabling our robot to run faster than it would have if powered on single channels instead. 
<img width="1000" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/eb8c9356-8c22-4b10-9c10-20d178e7dcc6">  

Per the wiring diagram, I soldered the appropriate inputs and outputs to achieve power-coupling before soldering the connections to the motors. I ran one motor driver with pins 6 and 7 on the Artemis board and another with pins 12, and 13. These pins were chosen because they support sending PWM signals via AnalogWrite commands from the Artemis to the motors. 

### Powering Motor Drivers 
From the wiring diagram above, we powered both motor drivers/motors with a 3.7V Li-Ion battery separate from the battery used to power the Artemis. This was due to a couple of reasons but mainly to ensure isolatio of power systems. The motors may generate electrical or electromagnetic noise which may cause interference with the Artemis. Also, the motors usually require a higher voltage and current as compared to the Artemis and may even draw larger currents during start-up or high load conditions which may interfere with the power needs and general operation of the Artemis. 



