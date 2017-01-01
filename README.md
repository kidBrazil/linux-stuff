# Ubuntu for Quadcopter Tuning

The purpose of this repo is to collect information and tools related to FPV Drones / Quadcopters and their tuning / configuration.


## Enabling STM32F USB on Ubuntu

There are a couple of things to watch for here. One being permissions / groups and the other being switching off Modem.Manager.

### Add User to dialout Group

```sudo adduser (username) dialout```

### Follow Instructions contained on this Wiki Page

Betaflight Configurator Wiki - [https://github.com/betaflight/betaflight/wiki/Installing-Betaflight]

------

## Summary Of Instructions 

###Platform Specific: Linux

Linux requires udev rules to allow write access to USB devices for users. The command bellow will create a template rule for you.

```(echo '# DFU (Internal bootloader for STM32 MCUs)'
 echo 'ACTION=="add", SUBSYSTEM=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="df11", MODE="0664", GROUP="plugdev"') | sudo tee /etc/udev/rules.d/45-stdfu-permissions.rules > /dev/null```
 
 ---
 
 Now you need to find the real product id of your FC. Type in the command bellow and plug your FC in and out. It should print a line with the product id out.

 ```udevadm monitor --environment --udev | grep ID_MODEL_ID```

 Now update the entry in "/etc/udev/rules.d/45-stdfu-permissions.rules" accordingly. You can add more than one rule in the file. The default product id is the FC in bootloader mode. Then reload rules using:

 ```sudo udevadm control --reload-rules && udevadm trigger```

 You can then test the rule using when your FC is plugged in:

 ```udevadm test $(udevadm info -q path -n /dev/ttyACM0)```

 Ensure line "MODE 0664 /etc/udev/rules.d/45-stdfu-permissions.rules" is present

 This assigns the device to the plugdev group(a standard group in Ubuntu). To check that your account is in the plugdev group type groups in the shell and ensure plugdev is listed. If not you can add yourself as shown (replacing with your username):

 ```sudo usermod -a -G plugdev <username>```

 If you see your ttyUSB device disappear right after the board is connected, chances are that the ModemManager service (that handles network connectivity for you) thinks it is a GSM modem. If this happens, you can issue the following command to disable the service:

 ```sudo systemctl stop ModemManager.service```

 If your system lacks the systemctl command, use any equivalent command that works on your system to disable services. You can likely add your device ID to a blacklist configuration file to stop ModemManager from touching the device, if you need it for cellural networking, but that is beyond the scope of cleanflight documentation.

 If you see the ttyUSB device appear and immediately disappear from the list in Cleanflight Configurator when you plug in your flight controller via USB, chances are that NetworkManager thinks your board is a GSM modem and hands it off to the ModemManager daemon as the flight controllers are not known to the blacklisted
