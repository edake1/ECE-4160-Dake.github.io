<h1>LAB 1: THE ARTEMIS BOARD AND BLUETOOTH</h1>
This lab was divided into two parts. The first part of the lab was meant to help us familiarize ourselves with the Arduino IDE ad the Artemis board. The second part of the lab had to do with establishing communication between your computer and teh Artemis board through bluetooth. 

PARTS REQUIRED 
- 1 x SparkFun RedBoard Artemis Nano
- 1 x USB C-to-C or A-to-C cable

PRELAB
For the prelab, we installed the Arduino IDE. I already had a previous installation so I just updated it. Since we would be using the SparkFun RedBoard Artemis Nano board for the class, we needed to install board support packages within the Arduino IDE. For our board, we installed the <b>SparkFun Apollo3 Arduino Core</b> by using the Additional Board Manager URL feature in the Arduino IDE (This process was quite straightforward). Then, we proceeded to install the latest version of the SparkFun Apollo3 board package using the board manager tool in the IDE. 
After the SparkFun Artemis board definitions had been installed, I proceeded to select the "RedBoard Artemis Nano" in the list of SparkFun Apollo3 boards. 

Now, the right SparkFun Arduino Core and board packages have been installed, so we proceeded to test the board with some simple projects. 

Blink It Up! 
This example was more like a proof of concept as its implementation is very simple and can be immediately verified on the board. I loaded the "Blink" project in the Built-in Examples section in the Arduino IDE. 

- Implementation 
The code implementation for "Blink" involves setting the board's built-in LED pin to 'HIGH" and "LOW" with a delay of 1000 between them. Since this code is implemented in the void loop, the process repeats and we see the built-in LED blinking. 
After loading the "Blink" project, I first verified (hit the verify button) the code to ensure that it correctly compiles, then I hit the upload button to load the code onto the Artemis board. After loading is done, the LED (the light is blue in color) starts blinking. See below. 

PIC 

VIDEO 


```
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);  // turn the LED on (HIGH is the voltage level)
  delay(1000);                      // wait for a second
  digitalWrite(LED_BUILTIN, LOW);   // turn the LED off by making the voltage LOW
  delay(1000);                      // wait for a second
}
```

