# HERMESS Telemetry protocol (TM)
_Version 1.0.0_



## Description
The REXUS Service Module (RXSM) telemetry system provides a continuous downstream of data from the experiments to the
teams ground station softwares. This protocol defines the both the interface between the
HERMESS Signal Processing Unit (SPU) with the RXSM and the interface between the ground telemetry receiver and the
HERMESS ground station software (GSS). This is possible because the REXUS PCM telemetry system in between behaves
opaque to the experimenteer teams, meaning the respective experiment output is supposed to be equal to the
datastream on the UART output line provided by the organizers on site for each experiment.

The HERMESS TM capabilities include:

* Periodic SPU state transmission
* Text message transmission



## UART via USB interface
The interface for the HERMESS TM downstream will be provided by the on-site organizers and is expected to be either
an RS-232 or RS-232 via USB interface. In the former case, an RS-232 to USB converter must be used. The physical connection
between SPU and RXSM is described in the REXUS User Manual.

The UART interface has these parameters:

1. Simplex, receive only, asynchronous UART link
2. Baudrate 38400 bit/s
3. 1 Start and 1 Stop bit
4. No parity bit
5. 8 bit words
6. No handshaking
7. A 64 byte large dataframe is sent from the SPU approximately twice per second



## The HERMESS TM dataframe
All HERMESS TM dataframes are exactly 64 bytes long to prevent buffer overflows in the RXSM-TM system. The frame shown here
is transmitted twice a second regardless of the current SPUs state, unless the TM feature is specifically disabled by
configuration.

    +------+------+------+------+------+------+------+------+------+------+------+
    | SYN0 | SYN1 | FRID | STA0 | STA1 | PTST | TE00 | .... | TE55 | CSM0 | CSM1 |
    +------+------+------+------+------+------+------+------+------+------+------+
    B: 00     01     02     03     04     05     06    ....    61     62     63
    
                          FIG 1: HERMESS TM dataframe


### `00 SYN0` to `01 SYN1`: Synchronization bytes
The value `0x17F0` is always sent as a demarcation bit sequence.


### `02 FRID`: Frame ID
An unsigned value starting with 0 at system start or reset and increment with each transmitted HERMESS TM dataframe
and wraps around when reaching its maximum value of 255.


### `03 STA0` to `04 STA1`: SPU state
1. Bit 15 (MSB in `STA0`): 1 = Restart after Watchdog Trigger
2. Bit 14: 1 = LO asserted
3. Bit 13: 1 = SOE asserted
4. Bit 12: 1 = SODS asserted and currently recording data
5. Bit 11: 1 = Write protection asserted
6. Bit 10: 1 = Flash storage was cleared before SODS was asserted
7. Bits 9 to 1: **NOT YET DEFINED**
8. Bit 0 (LSB in STA1): First byte of PTST


### `05 PTST`: Partial timestamp
The system timestamp is 8 bytes long and only one byte is being sent with each transmission, beginning with the
most significant byte in a big endianess style. When the most significant byte is being sent, bit 0 in STA1 is set
to 1 and is 0 for all other transmissions. To assemble the current timestamp, a ground station software must
therefore collect 8 consecutive HERMESS TM dataframes, beginning with a frame having the LSB in STA1 set to 1.


### `06 TE00` to `61 TE55`: Text message
Sends an info, warning or error message to the Groundstation Software. Messages may be spread over several HERMESS TM
dataframes. No more than one messsage can be transmitted via a single HERMESS TM dataframe.
1. Up to 56 bytes of printable ASCII characters
2. 1 byte ASCII `'0'` for infos, `'1'` for warnings, `'2'` for errors
3. Null padding, if message finished or no new messages queued


### `62 CSM0` to `63 CSM1`: Checksum
A simple numeric, unsigned 16-bit, big endianess sum of the previous 31 16-bit pairs of this dataframe with bytewise
independend value wrap around.



## Bit error and dropout tolerance
Most data losses are expected to occur during lift-off and reentry. While the larger RXSM PCM telemetry dataframe, containing
data from all modules on the REXUS vehicle, has forward error correction, long and short drops must be expected when
evaluating the TM datastream.