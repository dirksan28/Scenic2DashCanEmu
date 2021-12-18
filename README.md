# A lowcost DIY CAN BUS emulator for the Renault Scenic 2 dashboard

## Features
- powers up your dashboard without the need of installation in the car
- large test sequence. Shows saved km/milage on startup. (see video) 
- standalone operation (only with charging adapter) possible
- if connected to a serial console: timestamp and message logging on screen
- build to test your cluster after and during repair
\
\
[![](./pics/RenaultScenic2CAN-BusEmulatorDIYonGitHub-low.gif)](https://vimeo.com/568058419 "CAN bus - Emulator in action - Click to watch on Vimeo!")

click to watch the full vid on Vimeo

## What you need
- an Arduino (Uno) a cheap clone also works ;-)
- a MCP2515 module (e.g. https://www.amazon.com/s?k=MCP2515)
- The Arduino IDE https://www.arduino.cc/en/software to compile and upload the sketch to the arduino. 

## Hardware
![](./pics/mcp2515-arduino.jpg)

On the MCP2515 board there is a jumper which is labeled J1.
To work properly, make sure it is closed so the CAN bus termination is enabled.

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

Hint: The dash takes about 4-5 amps peak current (filament heating, caps) when it starts up. Make sure your power supply and cabling is sufficient.

## Software
The arduino project can be compiled and uploaded via the Arduino IDE. Either clone this project or download it as a [ZIP-File](https://github.com/dirksan28/Scenic2DashCanEmu/archive/refs/heads/main.zip).
The code which runs on the arduino is based on [MCP_CAN_lib](https://github.com/coryjfowler/MCP_CAN_lib).
For convenience a copy of this library also is part of this project. Just put the can-library into the library folder of your Arduino IDE (on linux: ~/Arduino/libraries).

The Message-Sequence can be extended or modified by patching the following code fragments within the [canEmu.ino](./Arduino/canEmulator/canEmulator.ino "link to canEmu.ino") file:

### Code
```c
/**
 * the following stuct contains the messages which are send to the dash
 * for initialization
 * feel free to add or remove messages.
 * {duration, id, dlc {byte1, byte2, ... byte_dlc}}
**/
const struct msgStruct initMessages[] PROGMEM = {
   {10, 0x35d, 8, { 0x10, 0x03, 0x20, 0x00, 0x00, 0x00, 0x50, 0x00}}   //dash on
  ,{4, 0x60d, 8, { 0x00, 0x10, 0x00, 0x00, 0x27, 0x73, 0x21, 0x71}}    //reset displ state
...
};

/**
 * the following stuct contains the messages which are send to the dash
 * within a loop
 * feel free to add or remove messages.
 * {duration, id, dlc {byte1, byte2, ... byte_dlc}}
**/
const struct msgStruct messages[] PROGMEM = { //load into flash-memory (sram was to small)
  {1, 0x743, 8, {0x02, 0x10, 0xC0, 0x00, 0x00, 0x00, 0x00, 0x00}}  //enable indicators
  ,{3, 0x743, 8, {0x04, 0x30, 0x06, 0x20, 0xFF, 0x00, 0x00, 0x00}} //left ind. lights
  ,{3, 0x743, 8, {0x04, 0x30, 0x07, 0x20, 0xFF, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x01, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x07, 0x00, 0x00, 0x00}}
  ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x06, 0x00, 0x00, 0x00}}
  ...
  };
```
To see what the code is doing. Start the serial console of the Arduino IDE at 9600 Boud.

## Current state
The code was tested on a V5.14 dashbord and runs fine.

## Design principles
- keep it stupid simple
- even "non-arduino experts" should be able to use and set up the project easily
- keep dependencies as few as possible (e.g. libraries, source files, additional hardware, ...)
- adaptable and expandable even without knowledge of C / C ++

## Collection of ideas
- Read the CAN messages from an external SD card (thus support for different dashboards without having to compile the code)
- single step mode (possibly forwards and backwards) via an additional button
- PCB layout for a self containing, single board solution

## Known issues
- CAN message to activate the lightbulb symbol is currently unknown  ![](./pics/missingBulbInd.png)

## Further infos & links
- If you search for further information, **tips & tricks on how to rapair your Renault Scenic II dashboard** - take a look at https://www.digital-kaos.co.uk/forums/showthread.php/59335-repair-dashboard-scenic-2
- One of the most comprehensive dashboard repair video (unfortunately in french - but the subtitle function should do the trick) https://www.youtube.com/watch?v=UUcnZQbhVvc
