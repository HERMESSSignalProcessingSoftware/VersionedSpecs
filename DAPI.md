# Data And Programming Interface (DAPI)
_Version 0.0.1_



## Description
The DAPI is the interface between the groundstation software and the SPU on the ground.
Its capabilites include:

* Programming and debugging the microcontroller
* Programming the FPGA fabric
* Reading out stored measurements and metadata
* Clearing on-board storage
* Configuring the SPU
* Triggering ADC calibrations
* Reading out ADC calibration data



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
    |  |  |   1.2.1: DAPI command frames   |  |  |
    |  |  |  |--------------------------|  |  |  |
    |  |  |  | 1.2.1.1: DAPI prog frame |  |  |  |
    |  |  |  |--------------------------|  |  |  |
    |  |  |  |--------------------------|  |  |  |
    |  |  |  | 1.2.1.2: DAPI cali frame |  |  |  |
    |  |  |  |--------------------------|  |  |  |
    |  |  |  |--------------------------|  |  |  |
    |  |  |  | 1.2.1.3: DAPI data frame |  |  |  |
    |  |  |  |--------------------------|  |  |  |
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
1. Full duplex UART link
2. Baudrate 115200
3. 1 stop bit
4. No parity bit
5. 8 bit words


### 1.2.1 DAPI command frames
1. Communication is initiated by a _request_ issued by the host system (computer running the ground
station software)
2. Every request is answered by the SPU with a _response_
3. The DAPI has a low execution priority in the SPU, so the time to the response is varying.
4. Requests consist of
    1. `1` _command byte_: Every request starts with one request byte.
        - `0x01`: Read recorded data and metadata
        - `0x02`: Trigger ADC calibrations **NOT IMPLEMENTED YET**
        - `0x03`: Read ADC calibration data **NOT IMPLEMENTED YET**
        - `0x04`: Write ADC calibration data **NOT IMPLEMENTED YET**
        - `0x05`: Read SPU configuration data **NOT IMPLEMENTED YET**
        - `0x06`: Write SPU configuration data **NOT IMPLEMENTED YET**
        - `0xAA`: Clear on-board storage
    2. `N` _Frame bytes_: Depending on the issued command the associated frame must be sent next.
    If no frame is associated with the issued command, no Frame bytes shall be sent.
    3. `2` _End Of Tx Bytes_: The value `0x17F0` must be sent at every end of transmission.
5. Responses consist of
    1. `1` _repitition byte_: The repitition of the command.
    2. `4` _response size byte_: A binary unsigned 32-bit value representing the number of
    frames that are about to be sent by the SPU.
    3. `N` _Frame bytes_: Depending on the issued command, response Frame bytes will be transmitted
    here by the SPU. Contrary to request Frame bytes, multiple frames of the same type may be
    transmitted here.
    4. `1` _Success byte_: `0x0F` if operation succeeded, `0xF0` otherwise.
    5. `2` _End Of Tx Bytes_: The value `0x17F0` must be received at every end of response.
6. DO NOT SEND ANOTHER REQUEST, BEFORE THE PREVIOUSLY ISSUED COMMAND HAS BEEN ANSWERED BY THE SPU.
Failure to comply with this rule may result in buffer overflows, highly likely in deadlocks or
some other sort of undefined behavior. Therefore never send more than 64 bytes between as a
single command.


#### 1.2.1.1 DAPI prog frame
**NOT DEFINED YET**



#### 1.2.1.2 DAPI cali frame
**NOT DEFINED YET**



#### 1.2.1.3 DAPI data frame
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
