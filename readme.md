# Hardware setting for MCP2515 SPI to CAN
As shown in the figure below, when used with raspberry pi or CM4, we need to push the switch to the top, and when used with CB1, we need to pull the switch to the bottom.
<img src=Images/mcp2515_switch.png width="800" /><br/>

# Software Settings
## Work with raspberry pi
* Add the following to the raspberry pi `/boot/config.txt` file
    ```
    dtparam=spi=on
    dtoverlay=mcp2515-can0,oscillator=12000000,interrupt=24,spimaxfrequency=10000000
    ```
    and restart raspberry pi

## Work with CB1
1. If the OS image of v2.2.1 version is used, the following dtb files need to be updated for stable use. If the OS image after V2.2.1 is used, this operation is unnecessary.
    * Download [sun50i-h616-biqu.dtb](./sun50i-h616-biqu.dtb)
    * Replace `/boot/dtb/allwinner/sun50i-h616-biqu.dtb` in the OS with the downloaded [sun50i-h616-biqu.dtb](./sun50i-h616-biqu.dtb). <img src=Images/dtb.png width="800" /><br/>
2. Uncomment `overlays=mcp2515` in `/boot/BoardEnv.txt`. <img src=Images/BoardEnv.png width="800" /><br/> <img src=Images/mcp2515.png width="800" /><br/>


# Getting started
* Input the `dmesg | grep -i '\(can\|spi\)'` command to test whether MCP2515 has been connected normally after the restart is completed.<br/>
    The normal response should be as follows
    ```
    [ 8.680446] CAN device driver interface
    [ 8.697558] mcp251x spi0.0 can0: MCP2515 successfully initialized.
    [ 9.482332] IPv6: ADDRCONF(NETDEV_CHANGE): can0: link becomes ready
    ```
    <img src=Images/mcp2515_init.png width="800" /><br/>
* Input `sudo nano /etc/network/interfaces.d/can0` command to create a new file named `can0`, and write the following content
    ```
    auto can0
    iface can0 can static
        bitrate 500000
        up ifconfig $IFACE txqueuelen 1024
    ```
    Set the Canbus speed to 500K (consistent with the speed set in klipper), and set the `txqueuelen` to 1024 bytes. Save(`Ctrl + S`) and Exit(`Ctrl + X`) after modification, input `sudo reboot`to restart host

* Each micro-controller on the CAN bus is assigned a unique id based on the factory chip identifier encoded into each micro-controller. To find each micro-controller device id, make sure the hardware is powered and wired correctly, and then run:<br/>
    `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`<br/>
    If uninitialized CAN devices are detected the above command will report lines like the following:<br/>
    `Found canbus_uuid=0e0d81e4210c`<br/>
    set the correct ID number in `printer.cfg`<br/>
    ```
    [mcu EBB]
    canbus_uuid: 0e0d81e4210c
    ```
