# CAN Bus Documentation
The first part of this guide was sourced from: https://www.klipper3d.org/CANBUS.html

The katapult github repository is found here: https://github.com/Arksine/katapult

EBB42 documentation: https://github.com/bigtreetech/EBB/blob/master/EBB%20CAN%20V1.1%20(STM32G0B1)/EBB42%20CAN%20V1.1/BIGTREETECH%20EBB42%20CAN%20V1.1%20User%20Manual.pdf

Manta E3EZ repository: https://github.com/bigtreetech/Manta-E3EZ/tree/master

The remaining information was pieced together from accross the internet and BTT documentation.

# Get Raspberry Pi Ready
Before we get CanBoot going we need to prepare our klipper host, a Raspberry Pi CM4 in this case...


## Edit can0 file
The purpose of this section is to allow our CM4 to communicate across the can0 network. This is simply done by creating a file called can0.
The following code creates a file called "can0" using "nano" in the directory "/etc/network/interfaces.d/". Sudo is needed since we are in the root directory.
```
sudo nano /etc/network/interfaces.d/can0
```
Once you have created the file, an empty window will appear in which you will add the following lines of text. Just copy paste them in.
```
allow-hotplug can0
iface can0 can static
    bitrate 1000000
    up ifconfig $IFACE txqueuelen 128
```
After copy pasting, press "ctrl+o" then "enter" to save the file. Press "ctrl+x" to exit the file. Take note of the bitrate number being 1,000,000. We want to make sure this is consistent accross our can0 network.

## Edit config.txt file
In this section we want to activate the USB2.0 hub on our CM4. By default it is disabled to save power. We are going to use the USB ports later so we need to activate them. Enter the following command to open the "config.txt" file found in "/boot/".

```
sudo nano /boot/config.txt
```
Scroll to the bottom of the file using the arrow keys and add the following line of text to the bottom. This will active the USB ports.
```
dtoverlay=dwc2,dr_mode=host
```
Go ahead and reboot your CM4 by entering the following command.
```
reboot
```



# MCU (Control Board MCU)
We now need to make the firmware for the MCU on the control board, a Manta E3EZ in my case. For the Manta, this involved creating the firmware, then flashing it to the board via a micro SD card. Follow your specific boards guide to do this part but take not of the configuration when making the firmware below.

## Make firmware
When making the firmware, we want to make sure that we have the proper configuration so that our control board can act as a CAN node. Enter the following command to get started.

```
cd ~/klipper/
make menuconfig
```

For the Manta E3EZ I used the following configuration.

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32G0B1)
    Bootloader offset (8KiB bootloader)
    Clock Reference (8 MHz crystal)
    Communication interface (USB to CAN bus bridge (USB on PA11/PA12))
    CAN bus interface (CAN bus (on PB12/PB13))
    USB ids (not changed)
CAN bus speed: 1000000
()  GPIO pins to set at micro-controller startup (not changed)
```
Press "q" to quit and "y" to save the menuconfig.

Use the following command to generate the klipper firmware for the MCU.

```
make clean
make
```

## Install firmware
In this section, we install the firmware made in the last section onto our control board's MCU.

The firmware file we created is called "klipper.bin" and is found in "~/klipper/out". Use a program, I use WinSCP, to transfer the "klipper.bin" file to your desktop computer. If you're more tech savy then feel free to use scp or some other method. Once you have the "klipper.bin" firmware file on your computer, you need to follow your control boards firmware flashing guide. I'll have the Manta E3EZ way below.

Next we need to rename our "klipper.bin" file to "firmware.bin". The MCU will recognize the file name and know it needs to flash itself. Once renamed, move the file to a micro SD card. Eject the card from your computer. Power off your Manta E3EZ if it is currently powered and insert the SD card into the MCU SD card slot on the Manta E3EZ board. Power the board and wait about 10 minutes (it takes less time but there's no indication of progress) for the firmware to flash. After it is done, power off the board, and remove the SD card. The board should now be flashed with its firmware.

You should be able to see the MCU on the can0 network by entering the following command.
```
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
You should see an output like...
```
Found canbus_uuid=11aa22bb33cc, Application: Klipper
```
With the uuid being different for each device.


# EBB42
Now we need to get our EBB42 onto the can0 network.

## Make firmware
### klipper
First, we need to make the "klipper.bin" firmware file like we did in the last section. This time, we need to configure it for the EBB42.
```
cd ~/klipper/
make menuconfig
```
The following is the configuration I used for the EBB42 klipper firmware.
```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32G0B1)
    Bootloader offset (8KiB bootloader)
    Clock Reference (8 MHz crystal)
    Communication interface (CAN bus (on PB0/PB1))
(1000000) CAN bus speed
()  GPIO pins to set at micro-controller startup (not changed)
```
Press "q" to quit and "y" to save the menuconfig.

Use the following command to generate the klipper firmware for the EBB42. No need to do anything further with the firmware for now.



### katapult (Formerly CanBoot)
Now we need to get katapult setup on the EBB42. Katapult is a bootloader. The purpose of it is to give us a way to update the klipper firmware on the EBB42 without having to attach a usb cable (which as you'll see is a bit of a hastle). It will enable us to update the klipper firmware using the CAN line that we are working so hard to set up.

Go ahead and navigate to the pi's home directory.
```
cd ~
```
Now we are going to clone the katapult github to our pi.
```
git clone https://github.com/Arksine/katapult
```

Now that the katapult repository is living on our pi, we need to add an update manager to our moonraker.cfg file. This will keep katapult up to date on both our pi and our EBB42. Add the following to your moonraker.cfg file.

```
# katapult update manager
[update_manager katapult]
type: git_repo
origin: https://github.com/Arksine/katapult.git
path: ~/katapult
is_system_service: False
```
Now navigate to the katapult directory. We are going to make the katapult firmware much like we did the two klipper firmwares.
```
cd katapult
make menuconfig
```
Go ahead and use the following configuration.
```
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32G0B1)
    Build Katapult deployment application (8KiB bootloader)
    Clock Reference (8 MHz crystal)
    Communication interface (CAN bus (on PB0/PB1))
    Application start offset (8KiB offset)
(1000000) CAN bus speed
()  GPIO pins to set on bootloader entry (no change)
[*] Support bootloader entry on rapid double click of reset button
[ ] Enable bootloader entry on button (or gpio) state (no change)
[*] Enable Status LED
(PA13)  Status LED GPIO Pin
```
Press "q" to quit and "y" to save the menuconfig.

Use the following command to generate the katapult firmware for the EBB42.
```
make clean
make
```

## Install firmware on EBB42
Now we are going to install the firmware we just created onto the EBB42. We are going to install both using different methods. The first one is the most annoying, but isn't too bad anyway.

### Install katapult
Let's go ahead and get katapult installed on our EBB42. Since we've already created the firmware we just need to run a few commands. But first, we need to do some things with our board. If you haven't activated the USB ports yet, go ahead and do that now.

First, we want to add the small sized jumper to the 120 Ohm pins. This is going to stay this way indefinitely. I know, committment is scary.

Now we want to add a jumper to the vbus pins, this will allow us to power the EBB42 with just usb power. If you haven't already the **disconnect your CAN plug**. You don't want to have 12/24V power and usb power criss crossing. Make sure your Pi (the entire Manta E3EZ board in this case) is powered off. Connect a USB-A to USB-C cable (capable of data transmission) from the Pi to the EBB42. Now, hold the boot button on the EBB42 and power on the Pi. Then let go of the boot button. This will put us into the DFU boot mode. **If you're using an EBB42 v1.1, this can cause fires. Search the web, I won't get into it here...** You could also have just held the boot button and pressed the reset button. Or held the boot button and plugged in the USB-C after the Pi had been powered on. Now, enter the following command to check that the EBB42 has entered dfu mode and that the Pi can see it.

```
dfu-util -l
```
You should get an output similar to the following...
```
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Found DFU: [0483:df11] ver=0200, devnum=6, cfg=1, intf=0, path="1-1.3", alt=2, name="@Internal Flash   /0x08000000/64*02Kg", serial="206B32815841"
Found DFU: [0483:df11] ver=0200, devnum=6, cfg=1, intf=0, path="1-1.3", alt=1, name="@Internal Flash   /0x08000000/64*02Kg", serial="206B32815841"
Found DFU: [0483:df11] ver=0200, devnum=6, cfg=1, intf=0, path="1-1.3", alt=0, name="@Internal Flash   /0x08000000/64*02Kg", serial="206B32815841"
```
Now, we want to move the katapult firmware to the EBB42. We do that by issuing the following command.
```
dfu-util -a 0 -D ~/katapult/out/katapult.bin -s 0x08000000:mass-erase:force:leave
```
It should output something similar to the following...
```
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

dfu-util: Invalid DFU suffix signature
dfu-util: A valid DFU suffix will be required in a future dfu-util release!!!
Opening DFU capable USB device...
ID 0483:df11
Run-time device DFU version 011a
Claiming USB DFU Interface...
Setting Alternate Setting #0 ...
Determining device status: state = dfuIDLE, status = 0
dfuIDLE, continuing
DFU mode device DFU version 011a
Device returned transfer size 1024
DfuSe interface name: "Internal Flash   "
Performing mass erase, this can take a moment
Downloading to address = 0x08000000, size = 4484
Download        [=========================] 100%         4484 bytes
Download done.
File downloaded successfully
dfu-util: Error during download get_status
```
If you were successful, go ahead and unplug the EBB42 from the USB. **Don't forget to remove the VBUS jumper**. Go ahead and plug the CAN cable back in. After that, power your printer back on.

### Install klipper
If you've been following the guide closely and everything has worked well enough then you should be on the home stretch now. Just a few more commands. Enter the following to see if we can see all of the CAN nodes on our network.
```
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
You should get an output similar to the following.
```
Found canbus_uuid=97e284228de1, Application: Klipper
Found canbus_uuid=3a96aa35ffaf, Application: CanBoot
Total 2 uuids found
```
CanBoot (the previous name of katapult) is the katapult node on our EBB42. The Klipper node is the CAN node on our control board's MCU. Now we can install the klipper firmware onto our EBB42 using the can0 network. Run the following command, altering the uuid to match your CanBoot uuid.
```
python3 ~/katapult/scripts/flash_can.py -f ~/klipper/out/klipper.bin -i can0 -u 3a96aa35ffaf
```
You should get an output similar to the following.
```
Sending bootloader jump command...
Resetting all bootloader node IDs...
Attempting to connect to bootloader
Katapult Connected
Protocol Version: 1.0.0
Block Size: 64 bytes
Application Start: 0x8002000
MCU type: stm32g0b1xx
Verifying canbus connection
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 14 pages
Verifying (block count = 427)...

[##################################################]

Verification Complete: SHA = 3FB41182CF14BA6262D710E427DB93DB0D8E3732
Flash Success
```

Now klipper should be installed on the EBB42.

# printer.cfg
The final step is to configure our MCUs in the printer.cfg file.

Make sure to add the following sections, adjusting the uuid to the ones you found.
```
[mcu]
canbus_uuid: 97e284228de1

[mcu EBB42]
canbus_uuid: 3a96aa35ffaf
```
Restart klipper and you should be good to go!
