# SMN (System management network) Address map

## Description

The system management network enables the PSP to talk to other system components on the package.
It uses a distinctive address space and the PSP implements an interface to map those addresses
into the PSP memory space so the registers of various devices can be accessed.

This map documents accessed addresses and their purpose if possible.

## CCX source to target mapping

Each CCX has its own set of registers but one CCX can access the range of another CCX by adding the offset from the following table
(the block offsets are stored in global PSP memory):

    +-----------+------------+------------+------------+------------+------------+------------+------------+------------+
    | tgt CCX ->|      0     |      1     |      2     |      3     |      4     |      5     |      6     |      7     |
    +-----------|            |            |            |            |            |            |            |            |
    | src CCX   |            |            |            |            |            |            |            |            |
    |    v      |            |            |            |            |            |            |            |            |
    +-----------+------------+------------+------------+------------+------------+------------+------------+------------+
    |    0      |     0x0    | 0xf0000000 | 0xe0000000 | 0xd0000000 | 0xc0000000 | 0x30000000 | 0x20000000 | 0x10000000 |
    |    1      | 0x10000000 |     0x0    | 0xf0000000 | 0xe0000000 | 0xd0000000 | 0xc0000000 | 0x30000000 | 0x20000000 |
    |    2      | 0x20000000 | 0x10000000 |     0x0    | 0xf0000000 | 0xe0000000 | 0xd0000000 | 0xc0000000 | 0x30000000 |
    |    3      | 0x30000000 | 0x20000000 | 0x10000000 |     0x0    | 0xf0000000 | 0xe0000000 | 0xd0000000 | 0xc0000000 |
    |    4      | 0xc0000000 | 0x30000000 | 0x20000000 | 0x10000000 |     0x0    | 0xf0000000 | 0xe0000000 | 0xd0000000 |
    |    5      | 0xd0000000 | 0xc0000000 | 0x30000000 | 0x20000000 | 0x10000000 |     0x0    | 0xf0000000 | 0xe0000000 |
    |    6      | 0xe0000000 | 0xd0000000 | 0xc0000000 | 0x30000000 | 0x20000000 | 0x10000000 |     0x0    | 0xf0000000 |
    |    7      | 0xf0000000 | 0xe0000000 | 0xd0000000 | 0xc0000000 | 0x30000000 | 0x20000000 | 0x10000000 |     0x0    |
    +-----------+------------+------------+------------+------------+------------+------------+------------+------------+

Example:
    Suppose you're executing on CCX ID 0 and want to access the register at SMN offset 0x1c880 on CCX 7. From the table
    the block for CCX 7 starts at offset 0xf0000000 on CCX 0 which added to the offset 0x1c880 yields 0xf001c880 resulting
    in the final SMN address for that register.

Current state:
    It is not possible to access all SMN regions on a different CCX from the master PSP, most accesses seem to cause a system reset.
    Only the region for the VEK seems to be exempt from this rule as the SEV firmware programs all AES engines only from the master PSP.
    Slave CCXs seem to have no way to access regions in the SMN network except its own region.

## Address Map

This address map collects information gathered about various SMM addresses and their purpose where it could be infered.

Legend for Table:
- Region = The base address of the region in the SMN (per PSP)
- Size   = Size of the whole region in bytes
- WP     = Flag whether the region is write protected by default and requires SMU message {0x14, 0x1} to disable the write protection
 Write protection should be enabled afterwards using SMU message {0x14, 0x0} to prevent accidental writes.
 Writes to this region will cause a system reset when write protection is enabled.
    - + = Write protected by default
    - - = No write protection
    - ? = Write protection status unknown
- MPsp   = Flag whether the master PSP can access this region on another CCX without poking the owning slave PSP.
    - + = Master PSP has access
    - - = Master PSP has no access
    - ? = Status unknown

Legend for Register Description:
- ? = Unknown purpose
- x = Not used/Reserved
- e = Enable/Disable bit
- a = Some part of an address (see description)
- d = Some arbitrary data (see description)
- s = See functional description of the register for the purpose of this bit.

|   Region   | Size | WP | MPsp | Offset |  RegSz |                                  Description                                           |       Register description       |
|------------|------|----|------|--------|--------|----------------------------------------------------------------------------------------|----------------------------------|
| 0x0001c760 |   32 | ?  |  ?   |        |        | Seems to be related to memory protection slots, see psp_dev_smn_addr_0x0001c760_init   |                                  |
|            |      |    |      | 0x00   |  32bit | Register 0                                                                             | ???????????????????????????????? |
|            |      |    |      | ...    |  ...   | Register 1 - 7                                                                         | ...                              |
|------------+------+----+------+--------+--------+----------------------------------------------------------------------------------------+----------------------------------|
| 0x0001c880 |  128 | +  |  -   |        |        | Memory protection slots                                                                |                                  |
|            |      |    |      | 0x00   |  32bit | Slot 0: Start address of protected region X86PADDR[47:20] + 4 flags                    | aaaaaaaaaaaaaaaaaaaaaaaaaaaa???? |
|            |      |    |      | 0x04   |  32bit | Slot 0: End address (inclusive) of protected region X86PADDR[47:20] + 4 flags          | aaaaaaaaaaaaaaaaaaaaaaaaaaaa???? |
|            |      |    |      | 0x08   |  32bit | Slot 0: Control register (seen 0x600000a | 0x6000006)                                  | ???????????????????????????????e |
|            |      |    |      | 0x0c   |  32bit | Slot 0: Unused/Reserved (no access observed anywhere)                                  | xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
|            |      |    |      | ...    |  ...   | Slot 1 - 6                                                                             | ...                              |
|            |      |    |      | 0x70   |  32bit | Slot 7: Start address...                                                               |                                  |
|            |      |    |      | 0x74   |  32bit | Slot 7: End address...                                                                 |                                  |
|            |      |    |      | 0x78   |  32bit | Slot 7: Control register...                                                            |                                  |
|            |      |    |      | 0x7c   |  32bit | Slot 7: Unused/Reserved...                                                             |                                  |
|------------+------+----+------+--------+--------+----------------------------------------------------------------------------------------+----------------------------------|
| 0x00050a00 |  256 | +  |  +   |        |        | Programs the AES engine with the EKs for the individual ASIDs, first memory channel    |                                  |
|            |      |    |      | 0x00   | 128bit | ASID 0 EK                                                                              | dddddddddddddddddddddddddddddddd |
|            |      |    |      | 0x10   | 128bit | ASID 1 EK                                                                              | dddddddddddddddddddddddddddddddd |
|            |      |    |      | 0x20   | 128bit | ASID 2 EK                                                                              | dddddddddddddddddddddddddddddddd |
|            |      |    |      | ...    |  ...   | ASID 3 - 15 EK                                                                         | ...                              |
|------------+------+----+------+--------+--------+----------------------------------------------------------------------------------------+----------------------------------|
| 0x00054a00 |  256 | +  |  +   |        |        | Programs the AES engine with the EKs for the individual ASIDs, second memory channel   |                                  |
|            |      |    |      | 0x00   | 128bit | ASID 0 EK                                                                              | dddddddddddddddddddddddddddddddd |
|            |      |    |      | 0x10   | 128bit | ASID 1 EK                                                                              | dddddddddddddddddddddddddddddddd |
|            |      |    |      | 0x20   | 128bit | ASID 2 EK                                                                              | dddddddddddddddddddddddddddddddd |
|            |      |    |      | ...    |  ...   | ASID 3 - 15 EK                                                                         | ...                              |
|------------+------+----+------+--------+--------+----------------------------------------------------------------------------------------+----------------------------------|
| 0x00059834 |   32 | -  |  ?   |        |        | Contains maximum SOC temperature values (the biggest one out of the values is picked?) |                                  |
|            |      |    |      | 0x00   |  32bit | SOC Max Temp 0 (extracted value - 0x188) ?= max T in F (0x282 - 0x188 = 250F > ~121°C) | ssssssssssssssssssss???????????? |
|            |      |    |      | ...    |  ...   | SOC Max Temp 1 - 7                                                                     | ...                              |
|------------+------+----+------+--------+--------+----------------------------------------------------------------------------------------+----------------------------------|
| 0x03b10700 |   24 | -  |  ?   |        |        | SMU mailbox interface                                                                  |                                  |
|            |      |    |      | 0x00   |  32bit | Argument 0                                                                             | dddddddddddddddddddddddddddddddd |
|            |      |    |      | 0x04   |  32bit | Status/Response                                                                        | ssssssssssssssssssssssssssssssss |
|            |      |    |      | 0x08   |  32bit | Unknown (Argument 1?)                                                                  | ???????????????????????????????? |
|            |      |    |      | 0x0c   |  32bit | Unknown (Argument 2?)                                                                  | ???????????????????????????????? |
|            |      |    |      | 0x10   |  32bit | Unknown (Argument 3?)                                                                  | ???????????????????????????????? |
|            |      |    |      | 0x14   |  32bit | Message identifier                                                                     | dddddddddddddddddddddddddddddddd |


Deprecated (convert to new format):

SMN address       Region size  Description
0x0004971c               4
0x0102e814               4
0x02d01380               4     Used in handle_dev_irq_0xf_maybe_watchdog()
0x02dc4000              32     Plays a role in the flash config (maybe SPI speed slection as well?)
0x038105a0             <unk>   Plays a role in MCM communication, psp_mcm_master_to_slave_something()
0x03b10034               4     Plays a role in taking MP1 out of reset

## Functional description

### 0x0001c880: Memory protection slots

The DRAM controller offers 8 "memory protection" slots which can be used to revoke access from the x86 core (and possibly other entities).
They are programmed by the PSP. So far only slot 0 and 1 seem to be used. Slot 0 covers the SMM region while slot 1 is holding the TMR region
for SEV-ES when enabled by the host.
One slot is configured by three registers, the first one holds the 1MB aligned physical address of the start of the region to protect. For current
hardware which implements 48bits bits [47:20] of the physical address are used to program the register. The remaining 4 LSBs contain some flags which purpose is
unknown so far, they could configure some caching or maybe some additional properties of the region (like whether to return 0xff or 0x00 on a read and what to do with a write).
The second register holds the last remaining physical address still belonging to the protected region (inclusive) and like the first register holds only bits [47:20] + some
unknown flags in the 4 LSBs. If a single 1MB region should be protected The address bits of the start and end register are equal.
Register three seems to be some sort of control register where only the purpose of bit 0 is known. Bit 0 toggles the protection on (1) or off (0), the remaining function of the bits remain
unknown.

### 0x03b10700: SMU - System management unit

@todo Find out whether there is only one SMU or one per CCX too.

The SMU is mainly used for power management related tasks but it also controls write access to the System Management Network (SMN), at least
writes for certain SMN ranges cause a system reset without sending the proper SMU message beforehand.

Bit 0 of the Status/Response register indicates whether a request is currently pending. 1 means that a new request can be issued. The following is the usual
sequence in pseudo code to send a message to the SMU:

    while (0x03b10700[1] & 0x1 == 0);
    0x03b10700[1] = 0;
    0x03b10700[0] = Argument 0;
    0x03b10700[5] = idMsg;
    while (0x03b10700[1] & 0x1 == 0);
    if (0x03b10700[1] != 1)
        error;
    else
        success;
    status/response = 0x03b10700[0];

Messages found:
-  0x1 Unknown, issued by the PSP during initialization
    - uArg0: 3
-  0x4 Unknown, issued by the PSP during initialization
    - uArg0: Deduced by a called function
-  0x6 Unknown
    - uArg0: SMU firmware region [End]|Start[31:16]
-  0x9 Unknown
    - uArg0: 1
-  0x14 Controls SMN write protection for certain regions
    - uArg0: 1 - Disable write protection 0 - Enable write protection

Sources: https://fahrplan.events.ccc.de/congress/2014/Fahrplan/system/attachments/2503/original/ccc-final.pdf

### 0x00050a00, 0x00054a00: DRAM Controller AES engine

The DRAM controller has a builtin AES encryption/decryption engine for the SME and SEV features. Each CCD controls two memory channels and should therefore have two controllers.
The encryption key, of a VM for example, is tied to a specific ASID (0 is usually the host itself). The PSP programs the VEK into the corresponding slot of both DRAM controllers to activate
AES encryption for the given ASID. The key is fixed to a 128bit length. This range can be accessed from the master PSP without poking the slaves.

### 0x00059834: Maximum SOC temperatures

Acess was observed in the AR3B module in function dxioIfGetMaxSOCTemperature(). Looks like it contains 8 values which are in Fahrenheit and offset by 0x188, reading the values and
calculating the values gives around 120-121°C which looks reasonable. The function reads all 8 values and takes the biggest temperature returning it minus the offset.
