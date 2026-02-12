# Standard IO PCB for PPUC Project
This is the hardware for a standard IO card which was designed for the PPUC pinball project, but can be useful in other applications as well.
It is designed for being low cost and functional for experimental use.  
Not everything is tested nor does it fulfill EMC or any other specifications.  
Use at your own risk!
To make use of anything of this project a basic understanding of electronics and programming is necessary. Nothing of it is "plug and play". I'm surely not liable for any damage to assemblies, pinball machines or even persons.

## Picture of the Board
![PCB Pic](IO_16_8_1/PCB_V010_TH.jpg)

## Name
IO_16_8_1 as it has 16 inputs, 8 outputs and one special output.  
Actually the inputs can also be used as outputs (see later in this document).

## Power Supply
The IO card must be supplied by 5 V (+-0.5 V) for logic and inputs. Additionally nominal 48 V for high power outputs. As these outputs are open drain, the voltage can actually be up to 60 V. In the circuit diagram the voltage is called 48 V.

## Controller
A RP2040 is used for it's cost/performance ratio. It is the same controller that is used in the Raspberry Pi Pico.

## Communication Interfaces
* RS_485: main communication interface for controlling outputs and scan inputs. Usually connected to a host interface like a PC or Raspberry Pi or similar.
* USB C: for programming, debug and flashing
* Serial Wire: alternative for programming, debug and flashing
* IIC: the PCB can be populated with an QWIIC connector (normally not mounted). This can be used for connecting to other PCBs or experimental use. It uses the same UART as for the RS485.
* UART, I2C, SPI: instead of using the input pins for inputs they can also be used as an additional interface (see docu of RP2040 for pins and function). Speed up to 10 kHz should be possible (100 kHz for In_2 and In_3).

## Switches (on board)
* Reset: hardware reset for controller (RP2040)
* Boot: if active the board connects to a PC like an USB stick. Usually used for programming the code into the flash memory on the board.
* DIP-switches: usually used to select an address for RS485 (16 combinations)

## Inputs (and low power outputs)
The 16 inputs are used for reading the state of an external switch. The switch is normally connected to ground (GND). Internally the input is pulled up to 5 V by a 10k resistor.

All inputs can also be used as signal-outputs (if programmed accordingly). The output is internally pulled up by a 10k resistor to 5 V. A higher voltage can be applied externally (max. 30 V) but be sure to have your hardware and software! setup correctly as no protection for damage is provided in that case. A maximum of 2 mA can be sinked/sourced.

The inputs In_2 and In_3 are designed for higher speed (100 kHz). The Pull-Up can be soldered to other pads to be pulled to 3,3 V (instead of 5 V). An input capacitance can be added (soldered) for better noise immunity but lower speed. Since version 1.0.1 the caps are soldered by default and must be removed if a high-speed serial connection is needed.

The input terminals can also be "miss"used for a switch matrix. To do that (no matter if used as column or row) the input resistor (330 Ohm) must be short cut (or replaced by a zero Ohm type). Background: the 330 Ohm resistor protects the circuit if the terminal is used as an output and accidentally a short cut is done at the terminal. Unfortunately the low voltage value gets too high when used in a switch matrix (diode voltage plus voltage drop due to added up currents through the resistor). This happens when more switches are closed at the same time. Having said that, there is an exception: if you use the Terminals 13 to 16 (mid power) as your drivers (strobe signal) then you don't need to change anything. That gives you for example a 4x4 or 8x4 matrix. You can also always use the specially designed PCB "Hardware_IO_16x8_matrix" for your switch matrix.

The inputs In_13 to In_16 have an additional open drain output transistor that allows higher currents in case of being used as an output (up to 3 A at 25 °C ambient temperature). The outputs are internally pulled up by a 10k resistor to 5 V. A higher voltage (max. 30 V) can be applied externally but be sure to have your hardware and software! setup correctly as no protection for damage is provided in that case. Please be aware that the default state (before software takes over control) is "on". 
If used as input In_13 to In_16 feature a memory (similar to a relay self-holding function). If the input has ever seen a low signal (e.g. switch closed to GND) it will stay that way until it gets an active high signal from the RP2040 (make the corresponding pin an output with signal "high" and then make it an input again). If this behavior is not wanted you can lift pin 3 of Q13 to Q16, then you have an input/output like e.g. terminal 1 with static behavior but no additional output power.  
Why not keep it simple and make all inputs the same? To give the choice to use the terminals as outputs (mid-power) as in pinball use case, there are often too many inputs and a lack on outputs. Therefore an output transistor to 4 of the I/O pins were added to be able to switch more power. As an side effect if this terminal is configured as an input it has this memory or self-holding characteristic (once triggered stays triggered). This can be dealt with by software, so that it actually acts like a normal input. In the end you have the benefit of mid-power-output but the drawback that the software getting more complex for the use case of a "normal" input. If you use the PPUC software and you have configured the terminal as an input, it acts like a normal input and you don't have to care about it.

Here is an example what steps are needed for input configuration, e.g. for IN_14:
1. find the corresponding GPIO for IN_14: that is GPIO16 at the physical Pin 27 of RP2040
2. set GPIO16 as output and set it to "high".
3. configure the GPIO16 as an input (now it is an input that is "low active", therefore setup for a switch to ground)
4. if a low at the terminal IN_14 appears, the GPIO16 will be read as "low" and stays that way even if the switch on terminal IN_14 opens again (self-holding)
5. when you think it's time to reactivate the input ("arm the input"), then goto 2.

If you do 5. always right after the input read "low" than it looks like a normal input without self-holding characteristic.


## High Power Outputs
There are 8 high power outputs available. Usually they are connected to pinball solenoids or flashers. A recovery diode for the solenoids is placed on the pcb, so no need for diodes at the coils. The outputs are open drain and drive currents up to 20 A with a load connected to a voltage of up to 60 V. Be aware that the current is limited by the fuse F2. It will blow within about 1 s at 20 A. You might use a 10 A fuse if necessary.

It's recommended to use 2 wires (back and forth) for each solenoid. That improves the EMC a lot.

## Special Output
One special output is available for high speed signals. The voltage is 5 V, it is a push pull output that can drive a current up to 8 mA. It can be used to e.g. control a WS2812 LED strip.

## Recommendations
* Connect only one Flipper to a pcb if possible. The advantage is, that all the energy of the big capacitor is available for the flipper, which is usually the solenoid with the highest current. If you have more flipper fingers than PCBs then spread the flipper solenoids as even as possible across the PCBs (e.g. if you have 6 flipper fingers it's better to connect 2 per board instead of all 6 at one board).
* Wire related switches and solenoids for fast flip devices (e.g. flipper, bumper, sling, kick back) to the same pcb. Then you can archive the shortest possible time lag between activating the switch and firing the solenoid.
* RS485 termination: short JP2 at the end of the RS485 bus. There should also be a 120 Ohm resistor at the beginning of the bus (e.g. USB to RS485 adapter).
* RS485 biasing: short JP1 and JP3 exactly once at any point of the RS485 bus. Only do this if no bias resistors are installed on the USB to RS485 adapter. 
* It's recommended to use flipper coils for higher voltage (like 40 - 50 V) as they draw less current. Since Version 1.1.1 low resistance coils (designed for 24 V) should work as well. You should also change the fuse F2 to 10 A for that application.
* If the MOSFETs on the high power outputs are not available (or you want to use the big player brands) here are some alternatives (I haven't tested any of them, but they should work according to their specification): GL40N10A4 (seems to be the same as NCE0140KA), RD3P200SNFRA, NVD6824NL, IPD90N10S4L-06, LSGG10R085W3


