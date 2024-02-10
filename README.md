# LAB 1: THE ARTEMIS BOARD AND BLUETOOTH

This lab was divided into two parts. The first part of the lab was meant to help us familiarize ourselves with the Arduino IDE ad the Artemis board. The second part of the lab had to do with establishing communication between your computer and teh Artemis board through bluetooth. 

## PARTS REQUIRED 
- 1 x SparkFun RedBoard Artemis Nano
- 1 x USB C-to-C or A-to-C cable

## PART 1 
The objective of part 1 is to setup and become familiar with the Arduino IDE and the Artemis board. 

To get started, we installed the Arduino IDE. I already had a previous installation so I just updated it. Since we would be using the SparkFun RedBoard Artemis Nano board for the class, we needed to install board support packages within the Arduino IDE. For our board, we installed the <b>SparkFun Apollo3 Arduino Core</b> by using the Additional Board Manager URL feature in the Arduino IDE (This process was quite straightforward). Then, we proceeded to install the latest version of the SparkFun Apollo3 board package using the board manager tool in the IDE. 
After the SparkFun Artemis board definitions had been installed, I proceeded to select the "RedBoard Artemis Nano" in the list of SparkFun Apollo3 boards. 

Now, the right SparkFun Arduino Core and board packages have been installed, so we proceeded to test the board with some simple projects. 

### Blink It Up!  
This example was more like a proof of concept as its implementation is very simple and can be immediately verified on the board. I hooked up the Artemis board to my computer via the  USB C-to-C cable and loaded the "Blink" project in the Built-in Examples section in the Arduino IDE onto the board.  

#### Implementation 
The code implementation for "Blink" involves setting the board's built-in LED pin to 'HIGH" and "LOW" with a delay of 1000 between them. Since this code is implemented in the void loop, the process repeats and we see the built-in LED blinking. 
After loading the "Blink" project, I first verified (hit the verify button) the code to ensure that it correctly compiles, then I hit the upload button to load the code onto the Artemis board. After loading is done, the LED (the light is blue in color) starts blinking. See below. 

PIC 

VIDEO 

#### Sample Code in Void loop

```
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);  // turn the LED on (HIGH is the voltage level)
  delay(1000);                      // wait for a second
  digitalWrite(LED_BUILTIN, LOW);   // turn the LED off by making the voltage LOW
  delay(1000);                      // wait for a second
}
```

### Example4_Serial - Using the Serial monitor 
For this example, I navigated to "Examples for RedBoard Artemis Nano" and selected "Example4_Serial" under Apollo3. I verified the code and uploaded it onto the board. 
Initially, when I uploaded the code onto the board, and pulled up the Serial Monitor, "gibberish" was displayed in the monitor. To fix this, the TA suggested increasing the baud rate so I tried adjusting the baud rate (rate of data transmission across the serial channel). Initially, the baud rate was around 9600, so I bumped it up gradually till I settled on 115200 baud. At this baud rate, I finally got intelligible outputs. 

In the Serial Monitor, I was able to type some inputs and they were displayed directly in the Serial monitor after the outputs from uploading "Example4_Serial". See below for pics. 

PIC OF OUTPUT FROM SERIAL MONITOR

### Example2_analogRead - Testing the temperature sensor 
This example was meant to test the on-board temperature sensor. After verifying and uploading this code onto the board, I pulled up the Serial monitor to see the temperature values being read by the temperature sensor on the board. 

#### Test 1 - Finger on chip
To test the sensor, I decided to press my index finger on the chip to transfer heat and thus increase the temperature being read by the sensor. I did this for a while and did not observe an appreciable increase in the temperature readings. 

#### Test 2 - Blowing hot air on chip
In an attempt to observe an appreciable change in the temperature, I proceeded to blow hot air onto the board instead. After a few seconds of doing this, I observed a slight change in the readings. See below for before and after pics. 

Pic - before 

Pic - after 

From the pics above, you would observe that the temperature readings changed from "32- - -" to "33- - -" after hot air was blown onto the board. 

### Example1_MicrophoneOutput - Testing the microphone 
After uploading the code onto the board, I pulled up the serial monitor and observed the microphone frequency readings. Due to the ambient noise i the lab, the lowest frequency I observed had a magnitude of about 125. 
To test the microphone, I brought my mouth close to the board and started speaking. Immediately, I noticed appreciable changes in the frequency recorded. I wanted to record a higher frequency so I tried whistling too and the results were even more noticeable. By whistling, I was able to get the loudest frequency recorded to about 1258 from around 912 (through speaking). See below for microphone readings 

Before 

After 


## PART 2 
The purpose of part two of this lab is to establish communication between your computer and the Artemis board through the Bluetooth stack.

### Computer Setup 
#### Install Python
I already had a python installation so I just re-verified that it was at least version 3.7

#### Install Venv and using virtual environment
Here, I used <em>pip</em> to install <em>venv</em> for Python by running ```python3 -m pip install --user virtualenv```. I then navigated to my project directory and created a virtual environment. 
After the virtual environment was created, I activated it and proceeded to install the relevant python packages for the environment by running ```pip install numpy pyyaml colorama nest_asyncio bleak jupyterlab``` 

With the package installatio done, I followed the lab instructions for unzipping the codebase into my project directory and got jupyter lab running. 

To test the robot's sensors more effectively, I carried out the following tasks. 

### ECHO COMMAND 
This task involves sending string data from the computer (in jupyter lab) to the Artemis board, where the data sent is processed (on the Artemis side) such that the computer receives back an augmented version of the data. 

Implementation
In the ECHO case in my ble_arduino.ino file, I stored the string data that will be received from the computer and used that data to create a response that will be sent back to the computer. See code below

```
tx_estring_value.clear();
tx_estring_value.append("Robot says -> ");
tx_estring_value.append(char_arr);
tx_characteristic_string.writeValue(tx_estring_value.c_str());
Serial.println(tx_estring_value.c_str());
```
PIC- sample output 

In the code above, "Hey there!" is sent to the Artemis, "Robot says -> " is appended to get an augmented output which is received on the Python side when ```ble.receive_string(ble.uuid['RX_STRING'])``` is called. "Robot says -> Hey there!" is displayed in the serial monitor and also returned when the code above is called in jupyter lab. 

### GET_TIME_MILLIS
We want to be able to get timestamped messages from the Artemis board so being able to get the time in milliseconds is important. This task involves writing a string like "T:123456" to the string characteristic. 

#### Implementation
I created a <em>GET_TIME_MILLIS</em> command in my ble_arduino.ino file, used the <em>append</em> method to write "T:"  and the current time in milliseconds (i.e. 123456) to the EString, then packaged that entire string (i.e "T:123456") to be transmitted to the computer by using the <em>writeValue</em> method. See below for code snippet for GET_TIME_MILLIS command

```
tx_estring_value.clear();
tx_estring_value.append("T:");
tx_estring_value.append((int)millis());
tx_characteristic_string.writeValue(tx_estring_value.c_str());
Serial.println(tx_estring_value.c_str());
```





