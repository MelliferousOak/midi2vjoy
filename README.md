# midi2vjoy

## 1. Introduction

This software provides a way to use MIDI controllers as joysticks.

A typical real joystick usually has only a few axis. This is limited by the HID specification of joystick. However, often when one plays complex games many axis of control are needed. For example, in flight simulators, the throttle, mixture, propeller, etc. all need their own control axis that is not available on common joysticks.

MIDI controllers have many sliders, and is a good candidate for use in this scenario. A few things are needed to get that to work as game controllers. Fortunately, a software called vJoy (http://vjoystick.sourceforge.net) can emulate joysticks in the system. This software reads the input from MIDI controllers, and drives the virtual joysticks created by vJoy.

This code is available through "Simplified BSD License".

## 2. How to use it

### 2.1. Overview

I will use the M-Audio Xsession-Pro as an example to show you how to use the midi2vjoy software.

1. Download and install vJoy from http://vjoystick.sourceforge.net/.
2. Use the vJoyConf tool ("vJoy" ->"Configure vJoy") to define the virtual joysticks (detail in 2.2).
3. Install midi2vjoy software.
4. Run "midi2vjoy -t" to test the code generated by MIDI device. These codes will be used for the configuration, so midi2vjoy know this vjoy axis or button to drive then the MIDI device is used (detail in 2.3).
5. Write/edit the mapping configuration file (detail in 2.4).
6. Run "midi2vjoy -m *midi* -c *conf*", where midi is the midi device and conf is the configuration file.

### 2.2. Configure vJoy

We need to configure the virtual joysticks based on the MIDI device we like to use. Here I use M-Audio Xsession-Pro as example. Xsession-Pro has 17 sliders and 10 buttons, so I divide that into three virtual joysticks.

![xsessionpro](/readme_images/xsessionpro.png)

With that, we will need to configure three virtual joysticks with vJoyConf tool. They are configured as following.

- vJoy #1: Z, Rx, Ry, Rz, Slider, and 8 Buttons;
- vJoy #2: Z, Rx, Ry, Rz, Slider, Slider2, and 1 Button;
- vJoy #3: Z, Rx, Ry, Rz, Slider, Slider2, and 1 Button.

### 2.3. Test MIDI device

To test the MIDI device, just run "midi2vjoy -t". First the program will print out a list of available MIDI input devices for you to choose. You will need to select the MIDI you want to use. For my Xsession-pro, it is number 1.

When the device is opened, you can mode the axis and press the buttons. The MIDI code that are sent by the MIDI device will be displayed on the screen. Just write down the first number (usually 176 for axis and 144 for buttons), and the second number (which is the index of the axis or buttons). Do this for all the axis and buttons you like to use. Use ctrl-c to exit the program.

Now you have all the necessary information to write a mapping configuration file.

In the example shown below, I am testing the MIDI interface 1. The axis I moved sent (176, 12), and the button I pressed sent (144, 46).

![test_cmd](/readme_images/test_cmd.png)

### 2.4. Edit mapping configuration file

The mapping configuration file is a simple text file. All lines starts with "#" are comments and will be ignored by the parser.

Each mapping line has four numbers. This correspond to mapping an axis or button from MIDI to the corresponding control on the virtual joysticks. The first two numbers are those you recorded in the midi test step above.

- o_type: This is a for axis and b for buttons;
- m_type: This is the message status (the first number you recorded in the testing session for each control);
- m_control: This is the index (the second number you recorded in the testing session for each control);
- v_id: The ID of the vJoy device (1, 2, 3, etc.);
- v_number: The axis name or button number on the vJoy device ("Z" for Z axis, and 3 for Button3).

Below the my mapping configuration file for Xsession-Pro.

```
# Midi to vJoy translation
# The format is one line for each control in the format of
#       o_type, m_type, m_control, v_id, v_number, (v_mult)
# o_type is the output type: axis or button
# m_type is the MIDI message status e.g., 176 (slider), 144 (channel 0 note on), 128 (channel 0 note off) etc.
# m_control is the ID of the midi message e.g., 60 is C3 when referring to a note.
# The m_type and m_control value of each MIDI input can be found
# when running the program in test mode. Just push/move the control
# and watch the messages showing up on the screen.
# v_id is the vJoystick ID where the MIDI message is translated to.
# v_number is the axis or button number MIDI message is contolling.
# v_mult is only needed for axes, and is a multiplier for the velocity, to allow us to saturate the axis input without having to play at full volume for every input
# The axis may be 'X', 'Y', 'Z', 'RX', 'RY', 'RZ', 'SL0', or 'SL1'.

A 176	12	1	Z	1.0
A 176	11	1	RX	1.0
A 176	14	1	RY	1.0
A 176	15	1	RZ	1.0
A 176	17	1	SL0	1.0
B 144	46	1	1
B 144	43	1	2
B 144	70	1	3
B 144	58	1	4
B 144	56	1	5
B 144	57	1	6
B 144	69	1	7
B 144	59	1	8

A 176	24	2	Z	1.0
A 176	25	2	SL0	1.0
A 176	26	2	SL1	1.0
A 176	27	2	RX	1.0
A 176	28	2	RY	1.0
A 176	29	2	RZ	1.0
B 144	44	2	1

A 176	31	3	Z	1.0
A 176	32	3	SL0	1.0
A 176	33	3	SL1	1.0
A 176	34	3	RX	1.0
A 176	35	3	RY	1.0
A 176	36	3	RZ	1.0
B 144	45	3	1

```

Once you have the configuration file, just run "midi2vjoy -m midi -c conf" to enjoy.

In my case, I will run:

```
midi2vjoy -m 1 -c xsessionpro.conf
```

