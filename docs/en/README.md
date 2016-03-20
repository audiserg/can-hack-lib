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

Protocol CanHacker (lawicel) described in [CanHacker for Windows documentation](http://www.mictronics.de/projects/usb-can-bus/)

Library implements it partially. [Suported commands listed here](protocol.md).
