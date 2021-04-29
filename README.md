# A lowcost DIY CAN emulator for the Scenic II dash

[![Little red riding hood](./pics/vidThumb.jpg)](https://vimeo.com/512398946 "CAN-Emulator in action - Click to Watch!")

## What you need
- an Arduino (Uno) a cheap clone also works ;-)
- a MCP2515 module (e.g. this one https://www.amazon.com/-/de/dp/B08HM...336095&sr=8-11)
- The Arduino IDE https://www.arduino.cc/en/software to compile and upload the sketch to the arduino. 

## Hardware
![](./pics/mcp2515-arduino.jpg)

On the MCP2515 board there is a jumper which is labeled J1. To function properly make sure it is closed so the CAN-Bus termination is enabled.

### Wireing
| PIN on Arduino  | PIN on MCP2515 CAN Bus Breakout Board  |
| ------------ | ------------ |
|D2|INT|
|D10|CS|
|D11|SI|
|D12|SO|
|D13|SCK|
|5V |VCC|
|GND|GND|

![](./pics/wireing.jpg)

|PIN# (grey connector on dash)| meaning|
| ------------ | ------------ |
|1|+12V|
|2|GND|
|29|CAN LOW|
|30|CAN HI|

## Software
The arduino project is attached to this post and can be compiled and uploaded via the Arduino IDE.
The code which runs on the arduino is based on [ MCP_CAN_lib]([https://github.com/coryjfowler/MCP_CAN_lib](https://github.com/coryjfowler/MCP_CAN_lib)
A copy of this library (and the full source code for the emulator) can be found within the provided zip file. Just put the can-library into the library folder of the Arduino IDE.

The Message-Sequence can be extended or modified by patching for the the following within the [canEmu.ino](./Arduino/canEmulator/canEmu.ino "link to canEmu.ino") file:
### Code
```c
...
const struct msgStruct messages[] PROGMEM = { 
  {20, 0x35D, 8, {0x90, 0x03, 0x00, 0x00, 0x00, 0x01, 0x50, 0x00}}
  ,{4, 0x743, 8, {0x02, 0x10, 0xC0, 0x00, 0x00, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x06, 0x20, 0xFF, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x07, 0x20, 0xFF, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x01, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x07, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x06, 0x00, 0x00, 0x00}}
  . . .
  };
```
