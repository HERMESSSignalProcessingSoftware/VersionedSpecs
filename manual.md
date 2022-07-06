# SPU handlers short manual
_Version 1.0.0_



## Description
This manual gives SPU handlers a very short guide on what general system
behaviour to expect from the SPU in its different firmware states.



## Data interfaces
The SPU provides two external data interfaces for non-development porpuses.

The Data and Programming Interface (DAPI) is connected via a mini USB plug and
enables ground operations such as reading out the measurements and configuring
the device. It also provides functionalities to flash and debug the SPUs firmware.
As such it is not available in flight or in an assembled payload configuration.

The telemetry (TM) downstream provides regular in flight status updates to ground
operators. This stream can be deactivated with the ground station software.



## System power up / Watchdog reset
A full ADC connectivity check, all ADC configurations and possible offset calibrations
are performed on SPU startup. The current firmware version and some other status
information alongside possible queued warning or error messages will be sent after
initialization.

A full system reset is performed, when the microcontroller subsystem detects a stuck
process for one second and issues a respective error message after restart.



## RXSM signaling lines
1. **Lift-Off (LO)**: Detected line changes are transmitted via DAPI and TM
2. **Start/Stop of Experiment (SOE)**: If configured and no set write protection, assertion
while LO and SODS are not set will initiate the reset of the measurement data storage.
3. **Start/Stop of Data Storage (SODS)**: Unless otherwise configured, will assertion reset
the measurement timestamp to 0 and start recording the measurements to the non-volatile
flash storage. Release will stop data acquisition, if the currently recorded time
has exceeded the configured minimum recording time. Data recording will stop regardless of
the SODS signal after exceeding the configured maximum recording time.



## LEDs
The SPU has 5 on-board SMD LEDs and 2 pluggable external LEDs. The latter ones are
connected to the eponymous on-board LEDs. The LEDs are labeled with shortcuts on the
PCB of the SPU.

1. **`POWER (PWR)`**: Powered with the internal voltage regulators and is always
on during operation.
2. **`FPGA LOADED (FL)`**: Powered with the FPGA internal negative reset and microcontroller
initialization finished line. This should be always on during operation.
3. **`Heartbeat Memsync (MS)`**: Because the name-bearing component was removed, it is
now illuminated when the write protection flag is not inserted.
4. **`Heartbeat Microcontroller Subsystem (MSS)`**: Toggles with 2Hz and indicates a running
microcontroller.
5. **`Recording (RC)`**: Toggles with 2Hz, when the measurement is active either for DAPI
live data acquisition or data storage after SODS assertion.



## Write protection
A write protection flag (a simple jumper) physically prevents the microcontroller from
writing or clearing measurement values from the flash storage unit. It should be placed
after the actual flight and prior to repowering the SPU.