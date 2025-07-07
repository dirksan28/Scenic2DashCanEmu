![Build Status](https://github.com/dirksan28/Scenic2DashCanEmu/actions/workflows/ci.yml/badge.svg)

# A low-cost DIY CAN bus emulator for Renault Scenic 2 dashboard repair
*My personal contribution to the right to repair initiative.*

## Features
- Build to test your Renault Scenic 2 dashboard during and after repair
- Powers up your dashboard without the need for installation in the car
- Large test sequence that shows saved km/mileage on startup (see video) 
- If connected to a serial console, timestamp and message logging on screen
- Supports standalone operation with a charging adapter

\
[![](./pics/RenaultScenic2CAN-BusEmulatorDIYonGitHub-low.gif)](https://vimeo.com/568058419 "CAN bus - Emulator in action - Click to watch on Vimeo!")

[click to watch the full vid on Vimeo](https://vimeo.com/568058419)


## What you need
- An Arduino (Uno), a cheap clone also works ;-)
- A MCP2515 module (e.g. https://www.amazon.com/s?k=MCP2515)
- The Arduino IDE to compile and upload the sketch to the Arduino (https://www.arduino.cc/en/software)

## Hardware
<img title="click to enlarge" src="./pics/wireing.jpg" width=50%>

### Wiring
#### Arduino <-> MCP2515
<img title="click to enlarge" src="./pics/mcp2515-arduino.jpg" width=50%>

| PIN on Arduino  | PIN on MCP2515 CAN Bus Breakout Board  |
| ------------ | ------------ |
|D2|INT|
|D10|CS|
|D11|SI|
|D12|SO|
|D13|SCK|
|5V |VCC|
|GND|GND|

On the MCP2515 board, ensure the jumper labeled J1 is closed to enable CAN bus termination.

#### The dashboard
|PIN# (grey connector on dash)| meaning|
| ------------ | ------------ |
|1|+12V|
|2|GND|
|29|CAN LOW|
|30|CAN HI|

> [!IMPORTANT]
>Ensure your power supply and cabling can handle 4-5 amps peak current (filament heating, capacitors) at startup.
>Especially when using small current limiting lab power supplies this may become a thing.
>Should your power supply not provide enough current, the program might run, but the dashboard will remain dark.

## Software
The Arduino project can be compiled and uploaded via the Arduino IDE. Either clone this project or download it as a [ZIP-File](https://github.com/dirksan28/Scenic2DashCanEmu/archive/refs/heads/main.zip).
The code running on the Arduino is based on [MCP_CAN_lib](https://github.com/coryjfowler/MCP_CAN_lib).
For convenience, a copy of this library is part of this project. Just place the entire can-library folder (MCP_CAN_lib-master) into the library folder of your Arduino IDE (on linux: ~/Arduino/libraries), (on MAC-OS: /Users/YOUR-USER-NAME/Arduino/libraries/) .

The message sequence can be extended or modified by editing the following code fragments within the [canEmu.ino](./Arduino/canEmulator/canEmulator.ino "link to canEmu.ino") file:

### Code
```c
/**
 * the following struct contains the messages which are send to the dashboard
 * for initialization
 * feel free to add or remove messages.
 * {duration, id, dlc {byte1, byte2, ... byte_dlc}}
**/
const struct msgStruct initMessages[] PROGMEM = {
   {10, 0x35d, 8, { 0x10, 0x03, 0x20, 0x00, 0x00, 0x00, 0x50, 0x00}}   //dash on
  ,{4, 0x60d, 8, { 0x00, 0x10, 0x00, 0x00, 0x27, 0x73, 0x21, 0x71}}    //reset displayl state
...
};

/**
 * the following struct contains the messages which are send to the dash
 * within a loop
 * feel free to add or remove messages.
 * {duration, id, dlc {byte1, byte2, ... byte_dlc}}
**/
const struct msgStruct messages[] PROGMEM = { //load into flash-memory (sram was to small)
  {1, 0x743, 8, {0x02, 0x10, 0xC0, 0x00, 0x00, 0x00, 0x00, 0x00}}  //enable indicators
 ,{3, 0x743, 8, {0x04, 0x30, 0x06, 0x20, 0xFF, 0x00, 0x00, 0x00}}  //left ind. lights
 ,{3, 0x743, 8, {0x04, 0x30, 0x07, 0x20, 0xFF, 0x00, 0x00, 0x00}}
 ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x01, 0x00, 0x00, 0x00}}
 ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x07, 0x00, 0x00, 0x00}}
 ,{1, 0x743, 8, {0x04, 0x30, 0x01, 0x20, 0x06, 0x00, 0x00, 0x00}}
  ...
  };
```
> [!NOTE]
>To see what the code is doing, start the serial console of the Arduino IDE at 9600 baud.

> [!TIP]
>If you remove line #4 (`#define ENABLE_CANBUS`), output to the MCP2515 breakout board will be skipped.
>By doing so, the code runs fine within the famous and free [WOKI Online Arduino Simulator](https://wokwi.com/arduino/new?template=arduino-uno), making it easier for debugging purposes.

## Current state
The code was tested on a V5.14 dashboard and runs fine.

## Design principles
- Keep it simple.
- Ensure even non-Arduino experts can easily use and set up the project.
- Minimize dependencies (e.g., libraries, source files, additional hardware, etc.).
- Make it adaptable and expandable even without knowledge of C / C++.

## Collection of ideas
- Read the CAN messages from an external SD card (supporting different dashboards without compiling the code).
- Single-step mode (possibly forwards and backwards) via an additional button.
- PCB layout for a self-contained, single-board solution.

## Known issues
- The CAN message to activate the lightbulb symbol is currently unknown.
  <br><img title="click to enlarge" src="./pics/missingBulbInd.png" width=40%>

## Further infos & links
### Resources
- If you are looking for further information, **tips & tricks on how to repair your Renault Scenic II dashboard**, check a look at https://www.digital-kaos.co.uk/forums/showthread.php/59335-repair-dashboard-scenic-2
- Radovan Petković Raša shows the **systematic troubleshooting and successful repair** of a Scenic II dashboard with lots of tips and great advice. **The best video** on this topic online.  If you don't speak Serbian, turn on automatic subtitle translation. https://www.youtube.com/watch?v=_f3Z28OoKZQ  - Thank you, Radovan for the time and effort you put into this. Great work!
- A good repair video that also shows **how to desolder the large display properly** (unfortunately in French, but the subtitle function should help): https://www.youtube.com/watch?v=UUcnZQbhVvc

### How to power up the dashboard even without a CAN-bus emulator
>[!TIP]
>For an initial test on your desk, you can partially switch on the dashboard (only the small clock display) even without a CAN-bus emulator.

Connect the **gray connector** to your power supply as usual:
- Pin1 +12V
- Pin2 GND 

And on the **[red connector](./pics/Scenic2SteeringWheelRadioRemote.pdf "Schematic and usage of the red connector for the radio and steering wheel remote.")**
- Pin1 via a 1kΩ resistor to +12V
<img title="click to enlarge" src="./pics/Scenic2CheckClockDisp.png" width=20%>

This simulates turning on your car's radio, which causes the clock display to turn on.

### Forks and Variants
- Thanks to David Douard, there is now a fork based on **ESP32 + SN65HVD230 as CAN transceiver** available. Check https://github.com/douardda/Scenic2DashCanEmu for details.
