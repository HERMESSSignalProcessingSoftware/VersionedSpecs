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


### 1.2 UART interface
1. Full duplex UART link
2. Baudrate 19200
3. 1 stop bit
4. No parity bit
5. 8 bit words


### 1.2.1 DAPI command frames
1. Communication is initiated by a _request_ issued by the host system (computer running the ground
station software)
2. Every request is answered by the SPU with a _response_
4. Requests and responses consist of
    1. [Required] 1 _command byte_
    2. [Required for some commands] N _attribute bytes_
    3. !!! CRC


### 1.2.1.1 DAPI prog frame
**NOT DEFINED YET**



### 1.2.1.2 DAPI cali frame
**NOT DEFINED YET**



### 1.2.1.3 DAPI data frame
**NOT DEFINED YET**



## Example communications
