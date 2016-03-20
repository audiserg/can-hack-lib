# CanHacker (lawicel) CAN adapter on Arduino + MCP2515

Use that [Library](https://github.com/autowp/arduino-mcp2515) to communicate with MCP2515

## Library Installation

1. Install [MCP2515 Library](https://github.com/autowp/arduino-mcp2515)
2. Download the ZIP file from https://github.com/autowp/arduino-canhacker/archive/master.zip
3. From the Arduino IDE: Sketch -> Include Library... -> Add .ZIP Library...
4. Restart the Arduino IDE to see the new "canhacker" library with examples

Testes with Arduino Nano.
On Arduino Uno when works with CanHacker for windows have problem with too long boot period and losing first command

## Usage

Example

```
#include <can.h>
#include <mcp2515.h>

#include <CanHacker.h>
#include <CanHackerLineReader.h>
#include <lib.h>

#include <SPI.h>

const int SPI_CS_PIN = 10;
const int INT_PIN = 2;

CanHackerLineReader *lineReader = NULL;
CanHacker *canHacker = NULL;

void setup() {
    Serial.begin(115200);
    while (!Serial);
    SPI.begin();
    softwareSerial.begin(115200);

    Stream *interfaceStream = &Serial;
    
    canHacker = new CanHacker(interfaceStream, null, SPI_CS_PIN);
    lineReader = new CanHackerLineReader(canHacker);
    
    pinMode(INT_PIN, INPUT);
}

void loop() {
    canHacker->processInterrupt();
    lineReader->process();
}
```

## Protocol

Protocol CanHacker (lawicel) described in (CanHacker for Windows documentation)[http://www.mictronics.de/projects/usb-can-bus/]

Library implements it partially. Suported commands listed below.

### `C[CR]`

This command switches the CAN controller from operational in reset mode. The
controller is no longer involved in bus activities.

Command is only active if controller was set to operational mode with command `O` before.

Return: [CR] or [BEL]

### `L[CR]`

This command will switch the CAN controller in Listen Only mode. No channel open
command (`O`) is required after issuing `L`.

Use the close channel command `C` to return to reset mode.

Return: [CR]

### `Mxxxxxxxx[CR]`

Set acceptance code. This command works only if controller in reset mode.

xxxxxxxx = Acceptance Code in hexadecimal

Default value after power-up is 0x00000000 to receive all frames.

Return: [CR] or [BEL]

### `mxxxxxxxx[CR]`

Set acceptance mask. This command works only if controller is setup with command `S` and in reset mode.

xxxxxxxx = Acceptance Mask in hexadecimal

Default value after power-up is 0xFFFFFFFF to receive all frames.

Return [CR] or [BEL]

### `N[CR]`

Read serial number from device.

Return: Nxxxx[CR]

xxxx = Serial number in alphanumeric characters.

### `O[CR]`

This command switches the CAN controller from reset in operational mode. The controller is then involved in bus activities. It works only if the initiated with `S` command before, or controller was set to reset mode with command `C`.

Return: [CR] or [BEL]

### `riiiL [CR]`

This command transmits a standard remote 11 Bit CAN frame. It works only if
controller is in operational mode after command `O`.

```
iii - Identifier in hexadecimal (000-7FF)
L   - Data length code (0-8)
```

Return: [CR] or [BEL]

### `RiiiiiiiiL [CR]`

This command transmits an extended remote 29 Bit CAN frame. It works only if
controller is in operational mode after command `O`.

```
iiiiiiii - Identifier in hexadecimal (00000000-1FFFFFFF)
L        - Data length code (0-8)
```

Return: [CR] or [BEL]

### `Sn[CR]`

This command will set the CAN controller to a predefined standard bit rate.
It works only after power up or if controller is in reset mode after command `C`.

The following bit rates are available:

```
S0 - 10Kbps
S1 - 20Kbps
S2 - 50Kbps
S3 - 100Kbps
S4 - 125Kbps
S5 - 250Kbps
S6 - 500Kbps
S7 - 800Kbps
S8 - 1Mbps
```

Return: [CR] or [BEL]

### `tiiiLDDDDDDDDDDDDDDDD[CR]`

This command transmits a standard 11 Bit CAN frame. It works only if controller is in operational mode after command `O`.

```
iii - Identifier in hexadecimal (000-7FF)
L   - Data length code (0-8)
DD  - Data byte value in hexadecimal (00-FF). Number of given data bytes will be
checked against given data length code.
```

Return: [CR] or [BEL]

### `TiiiiiiiiLDDDDDDDDDDDDDDDD[CR]`

This command transmits an extended 29 Bit CAN frame. It works only if controller is in operational mode after command `O`.
```
iiiiiiii - Identifier in hexadecimal (00000000-1FFFFFFF)
L        - Data length code (0-8)
DD       - Data byte value in hexadecimal (00-FF). Number of given data bytes will be checked against given data length code.
```

Return: [CR] or [BEL]

### `V[CR]`

Read hardware and firmware version from device.
Return: Vhhff[CR]

```
hh - hardware version
ff - firmware version
```

### `v[CR]`

Read detailed firmware version from device.
Return: vmami[CR]

```
ma = major version number
mi = minor version number
```

### `Zv[CR]`

This command will toggle the time stamp setting for receiving frames. Time stamping is disabled by default. 

If time stamping is enabled for received frames, an incoming frame includes 2 more bytes at the end which is a time stamp in milliseconds.

The time stamp starts at 0x0000 and overflows at 0xEA5F which is equal to 59999ms.

Each increment time stamp indicates 1ms within the 60000ms frame.

### Incoming messages

All incoming frames are sent after successful receiving, optional with time stamp.

No polling is needed.

They will be sent in the following format:

11 Bit ID Frame
```
tiiiLDDDDDDDDDDDDDDDD[ssss][CR]
```

11 Bit ID Remote Frame
```
riiiL[ssss][CR]
```

29 Bit ID Frame
```
TiiiiiiiiLDDDDDDDDDDDDDDDD[ssss][CR]
```

29 Bit ID RemoteFrame
```
RiiiiiiiiL[ssss][CR]
```

```
r    - Identifier for Remote 11 bit frame
R    - Idenifier for Remote 29 bit frame
t    - Identifier for 11 bit frame
T    - Idenifier for 29 bit frame
i    - ID bytes (000-7FF) or (00000000-1FFFFFFF)
L    - Data length code (0-8)
DD   - Data bytes (00-FF)
ssss - Optinal time stamp (0000-EA5F)
```
