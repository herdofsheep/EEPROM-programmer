# 28C256 Arduino programmer

## Branched from [28C256](http://ww1.microchip.com/downloads/en/DeviceDoc/doc0006.pdf) Project
## Using [Arduino Uno](https://store.arduino.cc/products/arduino-uno-rev3?queryID=undefined)

# Hardware Requirements
The components required to build the programmer are the following:
* 1x [Arduino Uno](https://store.arduino.cc/products/arduino-uno-rev3?queryID=undefined)
* 1x [28C256 EEPROM](https://www.mouser.co.uk/ProductDetail/Microchip-Technology/AT28C256-15SU?qs=MAR%2F2X5XOp7W6WUPOcncFQ%3D%3D)  to program
* 2x [SN74HC595 Shift Registers](https://thepihut.com/products/74hc595-shift-register-3-pack)
* 1x [Breadboard](https://www.mouser.co.uk/ProductDetail/BusBoard-Prototype-Systems/BB830?qs=VEfmQw3KOauhPeTwYxNCaA%3D%3D&countryCode=GB&currencyCode=GBP)
* Some [22AWG wire](https://thepihut.com/products/solid-core-wire-spool-25ft-22awg-white)

Optional
* 1x [ZIF socket](https://thepihut.com/products/40-pin-zif-socket) will protect your EEPROM's notoriously fragile legs
* 3x [0.1µF Capacitors](https://thepihut.com/products/0-1uf-ceramic-capacitors-10-pack) (6.3V or higher) will make your programmer more reliable
* 1x [Wire Strippers](https://thepihut.com/products/wire-strippers) for breadboarding

# Open in VSCode

### Install [PlatformIO](https://platformio.org/) in [VSCode](https://code.visualstudio.com/)

```bash
git clone https://github.com/herdofsheep/EEPROM-programmer
```

```bash
code EEPROM-programmer
```

### Handle dependencies

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e .
```

# Circuitry

The 8 I/O EEPROM pins are driven by the Arduino digital pins `D5` to `D12`.

The 15 EEPROM address lines are driven through the use of two [74HC595](https://www.ti.com/lit/ds/symlink/sn74hc595.pdf?ts=1647597704824&ref_url=https%253A%252F%252Fwww.google.com%252F) 8-bit shift registers. 

The analog pins of the Arduino board (`A0` to `A5`) are used to operate the control pins of the EEPROM (WE, RE, OE) and the data and clock lines of the shift registers.

![alt text](assets/schematic.png)

# Flashing

Connect Arduino Uno via USB with a baud rate of 115200 as the default.

Once flashed with the firmware, 

the board is completely stand-alone and will listen to the serial port awaiting instructions. These can be provided manually by the user using the serial monitor of the Arduino IDE or, more conveniently, using the `python3` program provided in the `interface` folder.

### Serial commands
The programmer accepts two types of serial commands that directly encode a read/write operation of a memory chunk. The default maximum chunk is set to 256 bytes. All the command fields must be provided in `HEX` format. The firmware is not case sensitive and automatically strips the encoding format (e.g. the inputs `0xEA`, `0xea`, `EA`, `ea` should be equivalent).

To read a chunk of `SIZE` bytes of memory starting from the address `ADDR` the following command must be used:
```
R, ADDR, SIZE
```

To write a chunk of `SIZE` bytes to the memory starting from the address `ADDR` the following command must be used:
```
W, ADDR, SIZE, <DATA>
```
where `<DATA>` represents a list of comma-separated bytes of length `SIZE`. When the writing operation has been completed the programmer returns the `OE` (operation ended) message on the serial port.

### Command-line interface
The programmer can be controlled through a `python3` script that automatically implements some base features. To run the script the following requirements must be satisfied:

* `python3` interpreter (3.8.12)
* [`pyserial` library](https://pythonhosted.org/pyserial/) (3.5)
* [`tqdm` library](https://tqdm.github.io/) (4.62.3)

Please notice how the version indicated between brackets represents the version used during development and do not represent a strict requirement, other version may work as well.

#### How to use the interface
After connecting the programmer to the computer via USB, **without any EEPROM in the socket**, the program interface can be started running the command:
```
python3 programmer.py
```
If the connection is established successfully the program will invite you to insert the EEPROM in the socket and press enter.

Once this operation is completed the desired operation can be selected using the provided menu. At the moment the following operations are implemented:

* `Memory dump`: prints on screen the content of the whole memory in lines of 16 bytes encoded in `HEX` format.
* `Read from address`: read the byte value located at a given address
* `Write to address`: write a byte to a given address
* `Program from binary file`: set the EEPROM content using a binary file
* `Fill memory with byte`: set all the addresses to the same byte value

Once the desired operations have been completed the interface will ask you to remove the EEPROM chip and press enter. This will close the serial connection and terminate the program.

During all the operations the autocomplete function can be invoked using the `TAB` key.



