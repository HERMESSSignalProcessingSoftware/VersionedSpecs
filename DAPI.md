# Data And Programming Interface (DAPI)
_Version 1.0.0_



## Description
The DAPI is the interface between the groundstation software (GSS) and the signal
processing unit (SPU) on the ground.  
Its capabilites include:

* Programming and debugging the microcontroller
* Programming the FPGA fabric
* Reading out stored measurements and metadata
* Clearing on-board storage
* Configuring the SPU



## Specifications
    |--------------------------------------------|
    |              1: USB interface              |
    |                                            |
    |  |--------------------------------------|  |
    |  |         1.1: JTAG interface          |  |
    |  |--------------------------------------|  |
    |                                            |
    |  |--------------------------------------|  |
    |  |         1.2: UART interface          |  |
    |  |  |--------------------------------|  |  |
    |  |  |      1.2.2: GICD Commands      |  |  |
    |  |  |--------------------------------|  |  |
    |  |  |--------------------------------|  |  |
    |  |  |      1.2.3: SICD Commands      |  |  |
    |  |  |--------------------------------|  |  |
    |  |--------------------------------------|  |
    |--------------------------------------------|
    
              FIG 1: DAPI LAYER BLOCKS


### 1: USB interface
1. Physical connection via USB Type B socket on the SPU (USB Type B Plug required for
connection)
2. Fullspeed USB 2.0 conform
3. USB Vendor ID: `1514`  
USB Product ID: `2008`  
Manufacturer string: `Microsemi`  
Product string: `Embedded FlashPro5`
4. An external power supply connected to the SPU is required. The USB interface does
not consume bus power.


### 1.1: JTAG interface
1. The JTAG interface is being used for programming and debugging of the FPGA fabric
and the microcontroller subsystem
2. Managed by chip vendor softwares: Libero SoC and SoftConsole
3. No explicit implementation necessary for the HERMESS groundstation software


### 1.2 UART interface
1. Full duplex, asynchronous UART link
2. Baudrate 115200
3. 1 stop bit
4. Even parity bit
5. 8 bit words
6. CTS handshaking (Transmit only on CTS == LOW; Other handshaking lines are ignored)
7. Integer values as Big Endianness


### 1.2.1 DAPI communication overview
1. Communication can be initiated by both, the SPU and the GSS and is generally stateless.
2. Communication data sent by the GSS is labeled Groundstation initiated communication _GICD_,
whilst data sent from the SPU is called _SICD_
3. The DAPI has a low execution priority in the SPU, so response timings are varying.
4. _GICD_ consist of
    1. `1` _command byte_: Every request starts with one request byte.
        - `0x00`: Echo command
        - `0x01`: Get device status **NOT IMPLEMENTED YET**
        - `0x02`: Read recorded data and metadata **NOT IMPLEMENTED YET**
        - `0x03`: Start Live Data acquisition
        - `0x04`: Stop Live Data acquisition
        - `0x05`: Read SPU configuration data **NOT IMPLEMENTED YET**
        - `0x06`: Write SPU configuration data **NOT IMPLEMENTED YET**
        - `0xAA`: Clear on-board storage **NOT IMPLEMENTED YET**
    2. `N` _Content bytes_: Depending on the issued command the associated frame must be sent next.
    If no content is associated with the issued command, no content bytes shall be sent. `N` must
    not be greater than 61.
    3. `1` _Content demarcation byte_: The value `0x17` must be sent next.
    4. `8 - ((N+3) mod 8)` _Padding bytes_: The value `0x00` as many times as necessary to 
    enforce the entire data transmission to be a multiple of 8 bytes long.
    5. `1` _End Of Tx Byte_: The value `0xF0` must be sent at every end of transmission.
5. _SICD_ consist of
    1. `1` _command byte_: Every request starts with one request byte and is losely coupled
    to the _GICD_ _command bytes_.
        - `0x00`: Send string message
        - `0x01`: Send device status **NOT IMPLEMENTED YET**
        - `0x02`: Send recorded data and metadata **NOT IMPLEMENTED YET**
        - `0x03`: Send Live Data acquisition **NOT IMPLEMENTED YET**
        - `0x05`: Send SPU configuration data **NOT IMPLEMENTED YET**
        - `0xAA`: Clear on-board storage finished **NOT IMPLEMENTED YET**
    2. `N` _Content bytes_: Depending on the issued command, response Frame bytes will be transmitted
    here by the SPU. Contrary to request Frame bytes, multiple frames of the same type may be
    transmitted here.
    3. `1` _Success byte_: `0x0F` if operation succeeded, `0xF0` otherwise.
    4. `2` _End Of Tx Bytes_: The value `0x17F0` must be received at every end of response.


### 1.2.2 GICD commands

#### 1.2.2.1 `0x00`: Echo Command
Echo a string message back to the GSS. The _content bytes_ are defined as:
1. Maximally 60 bytes of ASCII encoded string.
2. 1 Byte ASCII `'0'` for infos, `'1'` for warnings, `'2'` for errors

#### 1.2.2.4 `0x03`: Start Live Data acquisition
Enables the live data acqusition mode of the SPU. After issuing this command expect
a string message and live data acquisition frames with a frequency of 2Hz from the SPU.
This command has no _command bytes_.

#### 1.2.2.5 `0x04`: Stop Live Data acquisition
Disables the live data acqusition mode of the SPU. After issuing this command expect
a string message. This command has no _command bytes_.


### 1.2.3 SICD commands

#### 1.2.3.1 `0x00`: Send String Message
Sends an info, warning or error message to the Groundstation Software. These will not be queued until
a GSS connected to the DAPI, but in timewise close proximity to the triggering event.
The _success byte_ always indicates a successfull operation. The _content bytes_ are defined as:
1. Any amount of printable ASCII characters
2. 1 Byte ASCII `'0'` for infos, `'1'` for warnings, `'2'` for errors

#### 1.2.3.4 `0x03`: Send Live Data acquisition
Sends a recently captures datapackage containing all measurement values from all ADCs and all STAMPs.
The _success byte_ always indicates a successfull operation.
The _command bytes_ are defined as:
1. 1 byte indicating the number of dataframes
2. 8 bytes timestamp for first dataframe reading in fractions of 250us since start of data acquisition.
3. For every dataframe:
    1. 1 byte Stamp ID labeled 0 through 5. Note that the order appears random.
    2. 1 byte error bitfield:
        - `0x01` AdcLagging: The strain gauge rosette ADCs of this STAMP were not in sync with each other.
        - `0x02` StampLagging: This STAMP was not in sync with the other STAMPs.
        - `0x04` NoNew: No new value was received for this ADC. Not that this is normal for most temperature
        readings.
        - `0x08` Overwritten: The microcontroller could not read the measurements before they were overwritten.
    3. 2 byte SGR1 measurement value as 16 bit signed integer.
    4. 2 byte SGR2 measurement value as 16 bit signed integer.
    5. 2 byte RTD measurement value as 16 bit signed integer.

#### 1.2.3.3 DAPI data frame
1. Due to the implementation of the DAPI, the corresponding frame size for the following command will be defined as:
    - `0x01`: 512 Byte per Frame 
    - `0xAA`: 0 Bytes per Frame
2. Frame definition for command `0x01`:
    - 512 byte will be transmitted, 8 measurements are stored in this frame (Page alignment on Page 74 of latest SED)
        - Note: SED page 74 only showes the idea, the offsets are wrong calculated. 
    - Page start marker and page counter at offset `0x00`
        - Content:`0x0F`, 3 bytes page counter
    - Offsets according to the base address of the page and the offset of the measurement, first measurement at `0x04`. 
        - `0x00`: Timestamp value 
        - `0x04`: STAMP 1 highest 32 bit 
        - `0x08`: STAMP 1 lowest 32 bit 
        - `0x0C`: STAMP 2 highest 32 bit 
        - `0x10`: STAMP 2 lowest 32 bit 
        - `0x14`: STAMP 3 highest 32 bit 
        - `0x18`: STAMP 3 lowest 32 bit
        - `0x1C`: STAMP 4 highest 32 bit 
        - `0x20`: STAMP 4 lowest 32 bit
        - `0x24`: STAMP 5 highest 32 bit 
        - `0x28`: STAMP 5 lowest 32 bit 
        - `0x2C`: STAMP 6 highest 32 bit 
        - `0x30`: STAMP 6 lowest 32 bit 
        - `0x34`: Status Register 1
        - `0x38`: Status Register 2
    - Correction of Page 74 of the SED 
        - `0x00`: Page marker 
        - `0x04`: Measurment 1
        - `0x3C`: Measurment 2
        - `0x78`: Measurment 3
        - `0xF0`: Measurment 4
        - `0x12C`: Measurment 5
        - `0x168`: Measurment 6
        - `0x1A4`: Measurment 7
        - `0x1E0`: Measurment 8


## Example communications
**NOT DEFINED YET**
