# HERMESS Telemetry protocol (TM)
_Version 0.0.1_


## Description
Using the RXSM telemetry system this protocol defines how the groundstation software
has to interpret the datastream of the SPU. There is no uplink. The protocol enables
these features:

* Live readout of measurements (all avialable STAMPS)
* Live readout of status information regarding the SPU states


## Protocol
### Technichal Interface:
- 38400 BAUD, 8n1
- 60 Bytes per frame 
### Frame definition
    typedef struct {
        uint32_t timestamp;
        uint32_t stamp11;
        uint32_t stamp12;
        uint32_t stamp21;
        uint32_t stamp22;
        uint32_t stamp31;
        uint32_t stamp32;
        uint32_t stamp41;
        uint32_t stamp42;
        uint32_t stamp51;
        uint32_t stamp52;
        uint32_t stamp61;
        uint32_t stamp62;
        uint32_t statusReg1;
        uint32_t statusReg2;
    }Telemmetry_t;
### Transmiting behaviour:
- Transmit when time is available
- Transmit one frame after an other 
#### Todo
- Make sure that no frame will be overwritten while transmitting.
**NOT DEFINED YET**
