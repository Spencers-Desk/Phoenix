# CAN Bus Documentation
https://www.klipper3d.org/CANBUS.html

## Edit pi files
Create file can0
```
sudo nano /etc/network/interfaces.d/can0
```
Add below to the file
```
allow-hotplug can0
iface can0 can static
    bitrate 1000000
    up ifconfig $IFACE txqueuelen 128
```

## make menuconfig
```
make menuconfig
```

```
USB to CAN adapter
CAN PB12/PB13
```
rename from klipper.bin to firmware.bin
copy to SD card
insert into motherboard
power off then power on the motherboard (power cycle)
Give the MCU time to flash the new firmware
Once flashed, poweroff
Remove SD card
poweron
```
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
Correct output (UUID will vary)
```
Found canbus_uuid=97e284228de1, Application: Klipper
Total 1 uuids found
```