# Recover Asus router via tFTP and CFE flash command

### Background
This repo is meant to help people which have issues recovering their routers via normal Rescue Mode which is provided by manufacturer.

The Asus RT-AC1200G+ router encountered corruption due to an interrupted upgrade procedure caused by a power outage, resulting in damage to the firmware partition.

Initially, diagnosing the issue and determining if the router could even boot to TFTP mode was uncertain. To investigate further, a USB to serial converter was connected to the UART port on the router to gain access to the serial terminal.

Upon inspection, it became evident that something was amiss during the boot process as checksums did not match. Despite this, accessing TFTP was achievable by holding down the reset button (located within a small hole) and power cycling the router.

The router was looking for tftp payload but it could not be found.

Router was brought to life by manual flash of .trx image by CFE command flash:

```
CFE> help flash
CMD: [help flash]

  SUMMARY

     Update a flash memory device

  USAGE

     flash [options] filename [flashdevice]

     Copies data from a source file name or device to a flash memory device.
     The source device can be a disk file (FAT filesystem), a remote file
     (TFTP) or a flash device.  The destination device may be a flash or eeprom.
     If the destination device is your boot flash (usually flash0), the flash
     command will restart the firmware after the flash update is complete

  OPTIONS

     -noerase     Don't erase flash before writing
     -offset=*    Begin programming at this offset in the flash device
     -size=*      Size of source device when programming from flash to flash
     -ctheader    Check header of CyberTAN
     -noheader    Override header verification, flash binary without checking-mem;Use memory as source instead of a device

                  flash -ctheader : flash1.trx  (upgrade code.bin/code2.bin)
                  flash -ctheader : flash1.trx2 (upgrade code.bin/code2.bin)
                  flash -noheader : flash1.trx  (upgrade linux.trx)

*** command status = 0
CFE>
```

![20240208_214752](https://github.com/LGaljo/Recover-Asus-router-via-tFTP/assets/3329168/7d218b0e-c07f-46de-b56f-505b90484be4)


## Step to recover router Asus RT-AC1200G+
### Set up serial connection to router and computer
1. Download [Putty](https://putty.org/) and extract it somewhere
2. Open your router and find UART ports
3. Connect GND, TX and RX pins to USB to serial converter
4. Open Putty, choose serial, COM port (find it in device manager), and baudrate 115200. Click open.
5. If everything is right, readable text should written in the console when router is booting

### Next step is to prepare TFTP server 
1. Download [Open TFTP Server](https://sourceforge.net/projects/tftp-server/)
2. Download firmware file for your router and extract it to `C:\OpenTFTPServer`
3. Rename your file to something short (for example `flash1.trx`)
4. Set your computer network card to static IP `192.168.1.2` (or something in range of 192.168.1.2-255) (google how to do it)
5. Open terminal as administrator and write `cd C:\OpenTFTPServer` and execute
6. Start TFTP server by executing `.\RunStandAloneMT.bat`
7. In one of the lines in the output, you should see line: `Listening On: 192.168.1.2:69`

### Flash yout router
1. Boot router to TFTP mode by holding reset button and power cycle
2. On console should say that tftp starting up
3. Pres Ctrl+C few times to land in CFE console (starts with CFE> )
4. Write `flash -noheader 192.168.1.2:flash1.trx flash1.trx` according to help written above, execute it
(-noheader means no header check, 192.168.1.2:flash1.trx is path to your computer and file on tftp server, second flash1.trx is partition to flash to)
5. The process should start and report no errors
6. When the last line states `done. <sth number of> bytes written` (see my output below), you can reboot your router by power cycling it

```
Reading 192.168.1.2:flash1.trx:
Flash Device=flash1.trx
TFTP Client.
- last blk -
Done. 15831040 bytes read
Download of 0xf19000 bytes completed
Write kernel and filesystem binary to FLASH
Programming...copysize=15831040, amtcopy=15831040
done. 15831040 bytes written
```

7. The console window should show successful system boot and router should be alive again.

I hope this guide helps someone to fix their device as well without falling to deep into googling this stuff :)
