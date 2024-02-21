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
<img width="300" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/ad161126-73be-4390-8f8b-8bff401434f5"> <img width="300" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/db199fcf-bf28-4d1d-8d0c-c9901595d1d4">

For hooking up multiple sensors, the QWIIC breakout board came in handly. The breakout board was first powerered via a QWIIC cable thus providing three extra connection points for other sensors. I connected both Time of Flight (TOF) sensors to the Artemis via two QWIIC connection terminals on the breakout board.  
<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/1c766f0b-6f3c-47b5-89fe-8595f95340d8">  

In the picture above, you can see the two time of flight sensors connected via the breakout board. 

### I2C Sensor Address 
When the first Time of Tlight sensor was connected to the board, its address was printed in the serial monitor after it was detected during scanning. Per the sensor manual, the address of the sensor is supposed to be <em>0X52</em> but the address displayed was <em>0X29</em>. There was no cause for alarm at this point as <em>0X29</em> was simply the default address of <em>0X52</em> bitshifted to the right. 
<img width="400" alt="image" src="https://github.com/edake1/ECE-4160-Dake.github.io/assets/74028493/a513ce7c-1c69-494f-9195-3ea142a945a1">

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

### Sensor placement 
In order to ensure the robot is able to accurately detect obstacles in its environment, sensor placement is important. I decided to hook up one sensor to the front of my car and hook up the other to the back of the car. This design decision was to make sure the robot can detect obstacles with reference to its rear and front positions. Another option could have been to hook up both to the front but in that case the only obstacle detection reference will be the front where two sensors are working together to provide accurate data. 

### Time of Flight Sensor Modes 
The TOF sensor has some flexibility in the mode for taking measurements based on the range needed. You can either set the distance ranging mode to short or long via the <em>setDistanceModeShort()</em> and <em>setDistanceModeLong() methods. I decided to go with the short distance mode because I figured if the robot is going to be performing stunts, I want it to be maximally sensitive and accurate. The long distance mode sacrifices sensitivity to ambient noise for greater range so the short distance mode was a better option in this case. 





