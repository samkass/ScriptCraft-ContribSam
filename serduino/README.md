# serduino
The serduino module uses a simplistic serial protocol between JavaScript and an Arduino to read and write certain
Arduino pins from ScriptCraft.  I had plans to replace the serial protocol with something more advanced, such as
the one used by "MyRobotLab" but have not yet done so.

TODO: The Usage instructions below assume CraftBukkit.  They will need updates in order to work with CanaryMod.

## About

 This module communicates with an Arduino board over a serial link.  The Arduino
 must be running the included serduino.ino program.  The program and this module
 define a protocol in which the value of 6 pins are sent in each direction, along
 with an optional string (for debugging or LCD display).

 The protocol is as follows
 Sends: D2|D3|D4|PWM9|PWM10|PWM11|STR|00 (1 byte each except STR, termiante with 00)
 Receives: D5|D6|D7|A0|A1|A2|STR|00 (1 byte each except STR, terminate with 00)

 Each pin is 1 byte (0-255).  Digital values (0 for LOW and >0 for HIGH) are sent for
 pins 2, 3, and 4, and PWM duty cycles for PWM pins 9, 10, and 11.  Digital values
 (0 or 255) are received for pins 5, 6, and 7, and analog values (0-255) are received
 for Analog pins A0, A1, or A2.  (To specify the analog pins in this class, just use the
 numbers 0, 1, and 2 in the read functions.)

 Note that the current Arduino code can only manage updates every 1-2 seconds, so you'll
 have to hold buttons down for a bit for them to take effect, and sensors may not
 immediately trigger actions.

## Usage

 This module can only be used if the separate `jssc.jar` file is
 present in the CraftBukkit classpath. To use this module, you should
 ...

 1. Download jssc.jar from <https://github.com/scream3r/java-simple-serial-connector/releases>
 2. Save the file to the same directory where craftbukkit.jar resides.
 3. Create a new launch script.
 On Windows, call it start-jssc.bat and edit it to include
 the following command...

 ```sh
 java -classpath jssc.jar;craftbukkit.jar org.bukkit.craftbukkit.Main
 ```

 If you're using Mac OS, create a new craftbukkit-jssc.command
 file and edit it (using TextWrangler or another text editor) ...

 ```sh
 java -classpath jssc.jar:craftbukkit.jar org.bukkit.craftbukkit.Main
 ```

 4. Execute the craftbukkit-jssc batch file / command file to start
 Craftbukkit. You can now begin using this module to send and receive
 messages to/from a serial-connected Arduino which is running the
 included "serduino.ino" program.

 Below are some examples for working with serduino...

 ```javascript
 var serduino = require('serduino');
 // Create a connection object
 var conn = serduino.connect();
 // Start the connection
 conn.start();
 // After a second or two to make sure it's going...
 // Send an initial command to let the Arduino know we're here
 conn.writeDigital(2,0);

 // Now the Arduino will send us regular updates, and we can set pins

 // Put an LED and 1K resistor in series across pin 2.  Turn it on with:
 conn.writeDigital(2,255); // Any number other than 0 will work...

 // Or turn it on when any redstone is activated in the world...
 events.blockRedstone(function(event){conn.writeDigital(2,event.newCurrent);});

 // Put a potentiometer on analog pin 0 and read its value with:
 conn.readAnalog(0);

 // Callbacks can also be installed so a function is triggered when a pin changes...
 // This example launches some fireworks at location (-292,0,0) when pin 5 goes HIGH:
 conn.onPinChanged(5,function(pin,value) {
			if (value) new Drone(-292,0,0,1,server.worlds.get(0)).firework();
		});
 // This example sets night/day according to an analog pin (eg. potentiometer, light meter):
 //  (It doesn't process 0's in case there's a flaky connection to the sensor)
 conn.onPinChanged(0,function(pin,value) {
			if (value != 0) server.worlds.get(0).time = 6000+value*12000/256;
		});
 ```
