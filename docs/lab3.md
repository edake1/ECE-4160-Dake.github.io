# LAB 3: TIME OF FLIGHT SENSORS

This purpose of this lab is to equip the robot with time of flight sensors to enable it judge distances accurately in order to avoid running into obstacles and perform stunts. 

<b>PARTS REQUIRED</b> 
- 1 x SparkFun RedBoard Artemis Nano
- 1 x USB C-to-C or A-to-C cable
- 2 x 4m Time of Flight Sensors
- 1 x QWIIC Breakout board
- 2 x QWIIC connectors 

## The Time of Flight Sensors 
The time of flight sensors used for this lab are two VL53L1X sensors and wiring them was fairly straightforward as we just needed to refer to the specifications sheet and connect the right wires on the QWIIC connectors to the right pins on the sensors. The pins on the sensors were labeled so this task involved connecting Vin, GND, SDA, SCL, and XSHUT pins to the designated pins.
<img width="295" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/ad161126-73be-4390-8f8b-8bff401434f5"> <img width="283" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/3f56b82a-e5b5-4474-9580-91e388a83602">
For hooking up multiple sensors, the QWIIC breakout board came in handly. The breakout board was first powerered via a QWIIC cable thus providing three extra connection points for other sensors. I connected both Time of Flight (TOF) sensors to the Artemis via two QWIIC connection terminals on the breakout board. 
<img width="789" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/1c766f0b-6f3c-47b5-89fe-8595f95340d8">
In the picture above, you can see the two time of flight sensors connected via the breakout board. 

### I2C Sensor Address 
When the first Time of Tlight sensor was connected to the board, its address was printed in the serial monitor after it was detected during scanning. Per the sensor manual, the address of the sensor is supposed to be <em>0X52</em> but the address displayed was <em>0X29</em>. There was no cause for alarm at this point as <em>0X29</em> was simply the default address of <em>0X52</em> bitshifted to the right. 
<img width="505" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/a513ce7c-1c69-494f-9195-3ea142a945a1">

The recorded sensor address of <em>0X29</em> identified the Time of Flight sensor on the I2C bus we run into a problem since we are using two TOF sensors. If the two sensors are connected to the board, they will have the same address. This will be problematic as we would not be able to distinguish between them and as such not be able to work with them independently. Of course, the solution to this problem is to find a way to give one of the TOF sensors a different address, so that's what we did. 

The implementation involved shutting down one of the TOF sensors, assigning a new address and turning it back on. The active-low XSHUT pin connected to one of the TOF sensors made this possible as we had to just set it to low to turn off the sensor. 
```
...
pinMode(SHUTDOWN_PIN, OUTPUT); 
digitalWrite(SHUTDOWN_PIN, LOW);
Wire.begin();
distanceSensor2.setI2CAddress(SENSOR2_ADDRESS);
...
```



