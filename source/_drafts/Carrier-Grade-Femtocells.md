---
title: Carrier Grade Femtocells
tags:
- femotocell
- cellular
- Alcatel-Lucent
- physical access
- root access
banner:
---
When consumer grade femotocells or small cells came into the market in 2009 it was a hackers dream. Not only could you get all the radios and software needed to connect to the cellular network all in one cost effective box but you had everything you needed to get devices in range to connect to your cell with little, if any, notification to the user. In 2013, at Black Hat, a group of hackers showed that by using a Samsung femotocell on the Verizon network they were able to record phone calls, capture and send SMS messages, and capture all data traffic going to and from the connected device. These cells, once breached, are a surveillant's dream. 

Fast forward to 2018 and femotocells for 4G networks are still available for home users but where they are going is far more interesting. With the advent of the small cell architecture for 5G networks with a range significantly smaller than that of a typical macrocell the opportunity for compromise goes up tremendously. So before we get too far down the line we should take a step back and see just how secure a carrier grade cell is today. 
# Physical Security
The device I am going to take a look at today is an Alcatel-Lucent 9364 Metro Cell that was manufactured and sold by Nokia in 2012. It's an outdoor rugged device that is designed to be deployed by a network operator. It's a wall or pole mounted unit with water tight connectors for both its Ethernet connection as well as its dual antennas. Power consumption runs about 25W with a transmit power of up to 250 mW at the actual antenna connector. It's powered by PoE so there is only a single cable required to deploy it in the field outside of the optional GPS antenna. It weights about 4 lbs and is approximately 10x10x2 inches in size. 

The cell is held together by <insert number of security screws here> and contains a physical security measure that is [similar to one seen in other small cell devices](https://www.edn.com/design/consumer/4439769/Teardown--Inside-a-3G-MicroCell). The unit has a "door switch" that, when actioned, dumps the security keys from the device making it less that useful for its intended purpose. The switch can be defeated by knowing its location and carefully opening the top case while depressing the switch. This can be achieved by carefully lifting the apposing side of the case and using a long thin tool to hold the switch down until a more permanent measure can be installed. 

One of the simplest methods to locating physical security measures before you even attempt to open a case is to check online to see if someone else has already done a physical tear down of the device you are working with. An [iFixit](https://www.ifixit.com) style tear down is unlikely for a carrier grade device but luckily for us the FCC often opens and photographs devices before they get FCC approval. In this case, we can just go check out the internal board photos of the cell before we even get a screwdriver. ![FCC photo of the AL 9364 board][FCCInternalDoorSwitch] 

As we can see there is an obvious switch on the board and one can only surmise its purpose is to ensure that the device remains closed during operation. Once the case is opened and the door switch is left closed we can safely hold the switch down with a binder clip. From this point forward we are free to poke and prod around the internals of the device without fear of losing the devices security keys. 

# Getting Full Control

From here, getting into the device is trivial. On the board itself there are a couple spots that could be console ports and with a little trial and error we are able to connect to the serial port with a standard FTDI serial cable. The baud speed is a standard 115200 and after connecting the serial cable and rebooting the device we can watch the full boot process and see where the device comes up to a login prompt. 

```
femto-1213320015 login:
```

From here, we are faced with a couple options. We could try and brute force an account on the cell or we could be a little more clever. Once again, we reboot the device and watch it boot up. This time, we look a little closer in the boot early boot process to see if we can step in before things really get going. 

    Build:  V2_AE_1
    RESET:  POWER ON

    BLST START
    SRAM:   BYPASS
    SDRAM4: BYPASS
    SDRAM3: BYPASS
    SDRAM2: BYPASS
    SDRAM1: BYPASS
    SDRAM0: BYPASS
    I-Cache, D-Cache and MMU are now Enabled
    SPI NOR:    PASS
    NAND:   PASS
    LED:    PASS
    ETH_SW: ethernetInit: Toggle the mii reset pin
    PASS
    CRYPTO: PASS
    BLST END


    INV Option: 0x30011001
    LED CONFIG: 1
    Crypto CSN: 0000003a08c2
    INV Version:    0x13
    Inventory MAC:  00:30:ab:2b:8f:e4
    Inventory Crypto SN:    0000003a08c2
    Inventory PQDN: dnsalias.net

    NOT DOING FACTORY RESET

    NAND:  NAND device: Manufacturer ID: 0x2c, Chip ID: 0xda (Micron NAND 256MiB 3,3V 8-bit)
    256 MiB
    SPI NOR: MX25L1635D 2048 KB

    In:    serial
    Out:   serial
    Err:   serial
    Setting MAC to 00:30:ab:2b:8f:e4
    marvell_init: Initialising Marvell 88E6061 ethernet switch.
    GL1 ATU CTRL:   6161
    MAC UL  CTRL:   007f
    MAC UL1 CTRL:   007f
    MAC UL1 CTRL:   107f
    MAC UL  VLAN:   0032
    MAC UL1  VLAN:  0031
    MAC ARM  VLAN:  0003
    Net:   pc302_emac
    Hit any key to stop autoboot:

While there are some interesting data points in this output (like the ```Inventory PQDN``` endpoint, for example) lets focus in on the boot process and how we can inject ourselves into it. Once we see the ```Hit any key to stop autoboot:``` timer we are in luck. We tap a key and now the cell has dropped us to a u-boot shell. Type in ```help``` and we are offered up a full list of commands we can run at this stage in the process. 

    ?       - alias for 'help'
    base     - print or set address offset
    bdinfo  - print Board Info structure
    bootm   - boot application image from memory
    bootp   - boot image via network using BootP/TFTP protocol
    cmp  - memory compare
    cp   - memory copy
    crc32    - checksum calculation
    cryptoDump - Dumps the config memory space
    dhcp    - invoke DHCP client to obtain IP/boot params
    dns_query   -                            
    ebiset - <pin> <val - 0 or 1>emac_mii_read - 
    emac_mii_write - 
    ethernetInit - Initialise the ethernet switch
    ethernetPhyDump <phyid> - Dumps thernet PHY registers
    exit    - exit script
    fbsrboot - Boots to fbsr software
    fsload  - load binary file from a filesystem image
    fswrite - write binary file to a filesystem image
    go      - start application at address 'addr'
    help    - print online help
    iminfo  - print header information for application image
    loop     - infinite loop on address range
    ls  - list files in a directory (default /)
    marvellInit <0/1> - 0 Normal, 1 factory mode
    md   - memory display
    miscDestroyPrivateMiscData - Clear private data from the Misc flash area
    mm   - memory modify (auto-incrementing)
    modCommit - Commit a pending u-boot
    modRun - Select and run next u-boot
    modRunLinux - Select and run the generic
    modShow - Show state of the MODsmtest    - simple RAM test
    io_muxing_show - Shows current gpio muxing config
    mw   - memory write (fill)
    nand     - NAND sub-system
    nboot    - boot from NAND device
    nm   - memory modify (constant address)
    norPartErase - Erase NOR Partition
    norPartShow - Show NOR Partition Table(s)
    ping    - send ICMP ECHO_REQUEST to network host
    printcrc - print values of crc protected variables
    printenv- print environment variables
    programFLASH- Program FLASH
    programIPL  - Program IPL
    programMOD - Program MOD1 or MOD2
    rarpboot- boot image via network using RARP/TFTP protocol
    reset   - Perform RESET of the CPU
    rm  - remove file. 
    run     - run commands in an environment variable
    saveenv - save environment variables to persistent storage
    setcrc - set value protected crc variables
    setenv  - set environment variables
    sf  - SPI flash sub-system
    sfsec - SPI Flash security related ops
    sspi    - SPI utility commands
    switch_reset- Initialise the ethernet switch
    test    - minimal test like /bin/sh
    tftpboot- boot image via network using TFTP protocol
    umount  - umount yaffs
    version - print monitor version
    wdogTimerKick - kicks the watchdog 
    wdogTimerPause - Pause watchdog timer
    wdogTimerResume - Resume watchdog timer
    wdogTimerSet - Change the interval for next h/w wdog kick
    wdogTimerStart - Start watchdog 
    yaffstrace  - switch on lots of trace

### Kicking the Dog
There are a lot of useful commands here but a few moments into reading them the device reboots and it goes back into normal boot process. What happened? The board has a built in watchdog timer that was not getting the input it needs while we were sitting on the u-boot menu. Reboot the cell again and pause the boot process. As we can see from the helpful command list above we can kick the watch dog by running a ```wdogTimerKick``` command but that only buys us about a minute before we have to run the command again to keep the device from rebooting. So, we run the ```wdogTimerPause``` command and now the watchdog is disabled while we check out the options we have here. 
### Root Access
While commands like ```cryptoDump``` are distracting here we focus on our goal of getting root on the device. The simplest method from here looks to be modifying the boot arguments of the cell so that it boots up and launches directly into a shell rather than asking for authentication. Much like single user mode on linux we can ask the cell to boot and allow us further access since we are physically at the device. To do this we run a couple of commands:
```
setenv othbootargs 'console=ttyS0,115200 init=/bin/sh'
run bootcmd
```
These commands do a few simple things. The ```setenv othbootargs``` portion of the command asks u-boot to update the ```othbootargs``` variable with the arguments we provided. By injecting our own arguments we can control the boot process and further our goal of getting root. ```console=ttyS0,115200``` ensures that we maintain our console access after u-boot kicks off the boot process and ```init=/bin/sh``` has the underlying system kick off a shell once the system is up. Once we have the variable updated we kick off the boot process with ```run bootcmd``` with bootcmd just being a larger array of commands that encompasses the updated ```othbootargs``` variable. 