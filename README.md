# Aries-MAX30100 and MAX30102

* Library Name : VEGA_MAX30102 by VEGA-Processor
 
This is a library for the MAX30102 pulse oximeter sensor SPECIFICALLY FOR USE WITH VEGA ARIES Boards

In this library we are using I2C protocol.

 * MAX30102 Datasheet: https://www.digikey.in/en/datasheets/maxim-integrated/maxim-integrated-max30102 
 * ARIES Pinmap: https://vegaprocessors.in/files/PINOUT_ARIES%20V2.0_.pdf

Check out the above links for mlx datasheet and ARIES Pinmap. 
 
 * Connections:
 * MAX30102     Aries Board
 * VIN      -   3.3V
 * GND      -   GND
 * SDA      -   SDA-0
 * SCL      -   SCL-0
 * You can use any SDA,SCL pins 
 * For connecting to port 1 (SCL 1 and SDA1) of aries board use the default variable TwoWire Wire(1) instead of TwoWire Wire(0);
 * Use 10K pull-up resistors while connecting SDA and SCL pins to MAX30100 sensor for accurate measurements.

Other Useful links:

 * Official site: https://vegaprocessors.in/
 * YouTube channel: https://www.youtube.com/@VEGAProcessors


# Arduino-MAX30100 and MAX30102

Adding support for the MAX30102 see: http://bobdavis321.blogspot.com/2021/02/working-with-max30100-and-max30102.html

I decided to fork the MAX30100 library and make it into a MAX30102 library.  It is not nearly as easy as changing the register numbers!  Here is what I have done so far:
1. Copy all src files and rename them as MAX20102_XXXX
2. Edit all src files contents to replace MAX30100 with MAX30102
3. Change device ID from 11 to 15
4. Change registers as follows:
    01->02
    02->04
    03->05
    04->06
    05->07
    06->09
    07->0A
    09->0C and 0D
    16->1F
    17->20
5. Create second LED2_PA register (addx 0D above)
6. Biggest issue:  MAX30100 has 16 bit analog converters fitting into 2 byte transfers.
                            MAX30102 has 18 bit analog converters fitting into 3 byte transfers!
Here is the solution (I think):
              .ir=(long)((buffer[i*6]<<16)|(buffer[i*6+1]<<8)|buffer[i*6+2]),
              .red=(long)((buffer[i*6+3]<<16)|(buffer[i*6+4]<<8)|buffer[i*6+5]) });

[![Build Status](https://travis-ci.org/oxullo/Arduino-MAX30100.svg?branch=master)](https://travis-ci.org/oxullo/Arduino-MAX30100)

Arduino library for the Maxim Integrated MAX30100 AND MAX30102 oximetry / heart rate sensor.

![MAX30100](http://www.mouser.com/images/microsites/Maxim_MAX30100.jpg)

## Disclaimer

The library is offered only for educational purposes and it is not meant for medical uses.
Use it at your sole risk.

## Notes

Maxim integrated stopped the production of the MAX30100 in favor of MAX30101 and MAX30102.
Therefore this library won't be seeing any further improvement, besides fixes.

*IMPORTANT: when submitting issues, make sure to fill ALL the fields indicated in the template text of the issue. The issue will be marked as invalid and closed immediately otherwise.*

## Hardware

This library has been tested with the MikroElektronika Heart rate click daughterboard:

http://www.mikroe.com/click/heart-rate/

along with an Arduino UNO r3. Any Arduino supporting the Wire library should work.

The only required connection to the sensor is the I2C bus (SDA, SCL lines, pulled up).

An example which shows a possible way to wire up the sensor is shown in
[extras/arduino-wiring.pdf](extras/arduino-wiring.pdf)

Note: The schematics above shows also how to wire up the interrupt line, which is
currently not used by the library.

### Pull-ups

Since the I2C interface is clocked at 400kHz, make sure that the SDA/SCL lines are pulled
up by 4,7kOhm or less resistors.

## Architecture

The library offers a low-level driver class, MAX30100.
This component allows for low level communication with the device.

A rather simple but working implementation of the heart rate and SpO2 calculation
can be found in the PulseOximeter class.

This high level class sets up the sensor and data processing pipelines in order to
offer a very simple interface to the data:

 * Sampling frequency set to 100Hz
 * 1600uS pulse width, full sampling 16bit dynamic
 * IR LED current set to 50mA
 * Heart-rate + SpO2 mode

The PulseOximeter class is not optimised for battery-based projects.

## Examples

The included examples show how to use the PulseOximeter class:

 * MAX30100_Minimal: a minimal example that dumps human-readable results via serial
 * MAX30100_Debug: used in conjunction with the Processing pde "rolling_graph" (extras folder), to show the sampled data at various processing stages
 * MAX30100_RawData: demonstrates how to access raw data from the sensor
 * MAX30100_Tester: this sketch helps to find out potential issues with the sensor

## Troubleshooting

Run the MAX30100_Tester example to inspect the state of your rig.
When run with a properly connected sensor, it should print:

```
Initializing MAX30100..Success
Enabling HR/SPO2 mode..done.
Configuring LEDs biases to 50mA..done.
Lowering the current to 7.6mA..done.
Shutting down..done.
Resuming normal operation..done.
Sampling die temperature..done, temp=24.94C
All test pass. Press any key to go into sampling loop mode
```

Pressing any key, a data stream with the raw values from the photodiode sampling red
and infrared is presented.
With no finger on the sensor, both values should be close to zero and jump up when
a finger is positioned on top of the sensor.


Typical issues when attempting to run the examples:

### I2C error or garbage data

In particular when the tester fails with:

```
Initializing MAX30100..FAILED: I2C error
```

This is likely to be caused by an improper pullup setup for the I2C lines.
Make sure to use 4,7kOhm resistors, checking if the breakout board in use is equipped
with pullups.

### Logic level compatibility

If you're using a 5V-based microcontroller but the sensor breakout board pulls SDA and SCL up
to 3.3V, you should ensure that its inputs are compatible with the 3.3V logic levels.
An original Atmel ATMega328p considers anything above 3V as HIGH, so it might work well without
level shifting hardware.

Since the MAX30100 I2C pins maximum ratings aren't bound to Vdd, a cheap option to avoid
level shifting is to simply pull SDA and SCL up to 5V instead of 3.3V.

### Sketchy beat frequency readouts

The beat detector uses the IR LED to track the heartbeat. The IR LED is biased
by default at 50mA on all examples, excluding the Tester (which sets it to 7.6mA).
This value is somehow critical and it must be experimented with.

The current can be adjusted using PulseOximeter::setIRLedCurrent().
Check the _MAX30100_Minimal_ example.

### Advanced debugging

Two tools are available for further inspection and error reporting:

* extras/recorder: a python script that records a session that can be then analysed with the provided collection of jupyter notebooks
* extras/rolling_graph: to be used in conjunction with _MAX30100_Debug_ example, it provides a visual feedback of the LED tracking and heartbeat detector

Both tools have additional information on the README.md in their respective directories.

## Tested devices

* Arduino UNO r3, Mikroelektronika Heart rate click (https://shop.mikroe.com/heart-rate-click)

This combination works without level shifting devices at 400kHz I2C clock rate.

* Arduino UNO r3, MAX30100 custom board with 4.7kOhm pullups to 5V to SDA, SCL, INT

As above, working at 400kHz

* Sparkfun Arduino Pro 328p 8MHz 3.3V, Mikroelektronika Heart rate click

Even if this combination works (MAX30100 communication), the slower clock speed fails to deliver
the required performance deadlines for a 100Hz sampling.

## Troubled breakouts

This breakout board: http://www.sunrom.com/m/5337

Has pullups on the Vdd (1.8V) line. To make it work, the three 4k7 pullups must be
desoldered and external 4.7k pullups to Vcc of the MCU must be added.
