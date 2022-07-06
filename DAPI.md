# Data And Programming Interface (DAPI)
_Version 1.1.1_



## Description
The DAPI is the interface between the groundstation software (GSS) and the signal
processing unit (SPU) on the ground.  
Its capabilites include:

* Programming and debugging the microcontroller
* Programming the FPGA fabric
* Reading out stored measurements and metadata
* Clearing on-board storage
* Configuring the SPU



## Overview of DAPI specifications
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
1. Physical connection via mini USB socket on the SPU (mini USB Plug required for
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
2. Baudrate 115200 bit/s
3. 1 stop bit
4. Even parity bit
5. 8 bit words
6. CTS handshaking (Transmit only on CTS == LOW; Other handshaking lines are ignored)
7. Integer values big endian encoded


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
        - `0x05`: Read SPU configuration data
        - `0x06`: Write SPU configuration data
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
        - `0x03`: Send Live Data acquisition
        - `0x05`: Send SPU configuration data
        - `0xAA`: Clear on-board storage status **NOT IMPLEMENTED YET**
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
This command has no _content bytes_.

#### 1.2.2.5 `0x04`: Stop Live Data acquisition
Disables the live data acqusition mode of the SPU. After issuing this command expect
a string message. This command has no _content bytes_.

#### 1.2.2.6 `0x05`: Read SPU configuration data
Reads the EEPROM configuration of the SPU. After issuing this command expect
a Send SPU configuration data. This command has no _content bytes_.

#### 1.2.2.7 `0x06`: Write SPU configuration data
Writes the SPUs EEPROM configuration. For the configuration to take effect, you must restart the SPU.
Expect a string message back from the GSS. For runtime effiency, the _content bytes_ will not be
sanity checked the by SPU. The _content bytes_ are
1. 16 bytes printable ASCII encoded string for the name name designator. Shorter names are left aligned
and padded with nulls. At least the first character must not be null.
2. 1 Initilization information byte:
    - Bit 7 (MSB): 1 = Initiate SGR ADC with self offset calibration
    - Bit 6: 1 = Initiate SGR ADC with system offset calibration. Should not be set, when Bit 8 is set.
    - Bit 5: 1 = Initiate RTD ADC with self offset calibration
    - Bit 4: 1 = Initiate RTD ADC with system offset calibration. Should not be set, when Bit 8 is set.
    - Bit 3: Always 0
    - Bit 2: 1 = Enable storage of measurements on SODS signal asserted
    - Bit 1: 1 = Enable clear measurements storage on SOE signal asserted
    - Bit 0 (LSB): 1 = Enable the use of the telemetry system
3. 1 Sgr mode information byte: Contents of the SYS0 register of the SGR ADCs as of the ADS114x
datasheet chapter 9.6.4.4.
4. 1 Rtd mode information byte: Contents of the SYS0 register of the RTD ADCs as of the ADS114x
datasheet chapter 9.6.4.4.
5. 8 bytes minimum data storage time after SODS trigger in 250us increments.
6. 8 bytes maximum data storage time after SODS trigger in 250us increments. Must be greater than the previous value.



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
The _Content bytes_ are defined as:
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
    
#### 1.2.3.5 `0x05`: Send SPU configuration data
Reads the EEPROM configuration of the SPU and sends it to the GSS. The _success byte_ always indicates a
successfull operation. The _content bytes_ are identical to the ones used to write the SPU configuration data.
