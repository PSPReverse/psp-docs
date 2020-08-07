# Logs and output from a Ryzen7 1800X running on an Asus Prime X370-Pro

This directory contains logs for an off chip BL run in  the emulator with proxy mode configured.
The hardware used was a Ryzen7 1800X on an Asus Prime X370-Pro mainboard.
The following command line was used to record the log using proxy mode:
```
./PSPEmu \
    --emulation-mode sys \
    --flash-rom ../../binaries/Ryzen/Asus/PRIME-X370-PRO/PRIME-X370-PRO-ASUS-3803.ROM \
    --cpu-profile ryzen7-1800x \
    --trace-log /tmp/log \
    --timer-real-time \
    --intercept-svc-6 \
    --emulate-single-socket-id 0 \
    --emulate-single-die-id 0 \
    --psp-proxy-addr tcp://localhost:1236 \
    --emulate-devices ccp-v5:flash:timer2 \
    --iom-log-all-accesses \
    --trace-svcs \
    --io-log-write ../../ryzen_1800X.iolog
```

The resulting files are contained in this directory, namely:
* `off-chip-bl.log`: The output of the off chip BL when intercepting the debug log syscall
* `psp-emu.tracelog.bz2`: Compressed tracelog recorded by PSPEmu
* `proxy-mode.iolog.bz2`: Compressed I/O log for replaying

## Replaying the I/O log

For replaying the I/O log you need the exact same firmware image version which was used when
recording the log. You can download the image from ASUS website [here](https://dlcdnets.asus.com/pub/ASUS/mb/SocketAM4/PRIME_X370-PRO/PRIME-X370-PRO-ASUS-3803.zip).
You'll have to strip the UEFI capsule header from the image file bfore it can be used.

After decompressing the I/O log from this directory PSPEmu can be invoked with the following command line
for replay:
```
./PSPEmu \
    --emulation-mode sys \                      # Start with the off chip BL
    --cpu-profile ryzen7-1800x \                # Select CPU profile
    --flash-rom PRIME-X370-PRO-ASUS-3803.ROM \  # The flash image from ASUS website
    --trace-log run.tracelog \                  # The log to generate
    --timer-real-time \                         # Emulated timers tick in realtime
    --emulate-devices ccp-v5:flash:timer2 \     # Emulate those devices instead of reading from the I/O log
    --iom-log-all-accesses \                    # Log all I/O accesses to the trace log
    --trace-svcs \                              # Trace all syscalls being made
    --intercept-svc-6 \                         # Intercept debug log syscall
    --dbg 4000 \                                # Enable GDB stub on port 4000
    --io-log-replay proxy-mode.iolog            # The I/O log to replay
```
