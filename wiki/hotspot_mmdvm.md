# Setting up an MMDVM M17 Hotspot from scratch

The following tutorial describes how to setup an M17-capable MMDVM hotspot from scratch on a fresh Raspberry Pi OS / Debian image.

## Essential elements of an MMDVM M17 Hotspot



Important to know it that a functioning MMDVM M17 system consists of three elements;

1. The **firmware** of the MMDVM hat, whose version must be equal/above 1.6.0
2. A recent version of the **MMDVMHost** program
3. The gateway program **M17Gateway** which connects the controller to the reflectors

## Updating the hotspot firmware

### Update your system and get tools for compiling

First we should update our operating system and get some tools for getting the sources and compiling them.

```bash
sudo apt-get update
```

```bash
sudo apt-get install git gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi
```

### Download the firmware as source code

The newest firmware can always be found on GitHub. 

```bash
cd ~
git clone [https://github.com/g4klx/MMDVM_HS](https://github.com/g4klx/MMDVM_HS)
cd MMDVM_HS/
git submodule init
git submodule update
```

### Prepare compiling / set flags to suit your hardware

Then you need to edit Config.h with the appropriate settings for your hardware:

```bash
nano Config.h
```

Select your board by uncommenting the correct line, in this case the MMDVM hat Rev. 1.1 up to 1.4 by DB9MAT/DF2ET and save with “STRG”+“O”

```
// Select one board (STM32F103 based boards)
// 1) ZUMspot RPi or ZUMspot USB:
// #define ZUMSPOT_ADF7021
// 2) Libre Kit board or any homebrew hotspot with modified RF7021SE and Blue Pill STM32F103:
//#define LIBRE_KIT_ADF7021
// 3) MMDVM_HS_Hat revisions 1.1, 1.2 and 1.4 (DB9MAT & DF2ET)
#define MMDVM_HS_HAT_REV12
// 4) MMDVM_HS_Dual_Hat revisions 1.0 (DB9MAT & DO7EN)
// #define MMDVM_HS_DUAL_HAT_REV10
// 5) Nano hotSPOT (BI7JTA)
// #define NANO_HOTSPOT
// 6) NanoDV NPi or USB revisions 1.0 (BG4TGO & BG5HHP)
// #define NANO_DV_REV10&amp;gt;
// 7) D2RG MMDVM_HS RPi (BG3MDO, VE2GZI, CA6JAU)
// #define D2RG_MMDVM_HS
// 8) BridgeCom SkyBridge HotSpot
// #define SKYBRIDGE_HS
// 9) LoneStar USB Stick ADF7071
// #define LONESTAR_USB
```

Please pay attention to the TCXO (check the right frequency, it's visible on the PCB) and uncomment the correct line

```
// TCXO of the ADF7021 
// For 14.7456 MHz: 
#define ADF7021_14_7456 
// For 12.2880 MHz: 
// #define ADF7021_12_2880
```

Select the way host and MMDVM communicate, in case of the Raspberry Pi's GPIOs, that's STM32_USART1_HOST

```
// Host communication selection:
#define STM32_USART1_HOST
//#define STM32_USB_HOST
//#define STM32_I2C_HOST
```

Activate LEDs for M17

```
// Use the D-Star and P25 LEDs for M17
// define USE_ALTERNATE_M17_LEDS
```

## Compiling

Now you are ready to compile with

```bash
make
```

Should you have compiled the firmware having used different options / flags before, don't forget to do a

```bash
make clean
```

before

```bash
make
```

to delete the previous code.

If everything compiles as expected, you are ready to flash.

However depending on what kind of Raspberry Pi / operating system you are using, you have to make sure that the serial port is is free and not in use for Bluetooth or console, or you will get an error message.

If you are using a Pi2 or Pi4 you may jump ahead to Flash the Firmware.

### Deactivate serial-getty

If you are using a new copy of Raspberry Pi Os on a PI3, Pi Zero W or Zero 2W, you need to deactivate serial-getty:

Edit /boot/cmdline.txt:

```bash
sudo nano /boot/cmdline.txt
(remove the text: console=serial0,115200)
```

Disable systemd services

```bash
sudo systemctl disable serial-getty@ttyAMA0.service
sudo systemctl disable bluetooth.service
```

Edit /boot/config.txt

```bash
sudo nano /boot/config.txt
```

Add the following lines at the end of /boot/config.txt:

```
enable_uart=1
dtoverlay=pi3-disable-bt
```

And restart your Pi

```bash
sudo reboot
```

### Flash the firmware

Having everything prepared, you are now ready to flash the firmware. Depending on what kind of hardware you are using, different commands exist to do the flashing:

like

```bash
sudo make mmdvm_hs_hat
```

or

```bash
sudo make mmdvm_hs_dual_hat
```

or

```bash
sudo make zumspot-pi
```

If flashing fails due to a missing stm32flash, you can install it manually with the following:

```bash
cd ~
git clone https://git.code.sf.net/p/stm32flash/code stm32flash
cd stm32flash
make
sudo make install
```

Now you can try again.

If you are using a dual hat, make sure the booting jumpers on the PCB are set correctly. 

```
BOOT0: completely removed
BOOT1: set to BOOT 1 -
```

If everything went as supposed to, you should have FW 1.60 on your hotspot now and you can proceed to setting up MMDVMHost.

## Install and configure MMDVMHost

### Optional: Install the OLED DRIVER

If you are planning to use an OLED-display, you should first enable i2c and spi and install the OLED driver in order to compile the OLED support into MMDVMHost.

Detailed information on how to proceed with this can be found on [F5UII's homepage.](https://www.f5uii.net/en/installation-oled-display-ssd1306-raspberry-pi-mmdvm-mmdvmhost/) In short you need to enable spi and i2c with

```bash
raspi-config
```

add

```
i2c-dev
spidev
```

to

```
/etc/modules
```

and download/compile/install the driver the known way via git.

```bash
git clone https://github.com/hallard/ArduiPi_OLED
cd ArduiPi_OLED
sudo make
```

### Download the MMDVMHost source code and compile & install it

Check out the source code via Git and change directory into it

```bash
git clone https://github.com/g4klx/MMDVMHost
cd MMDVMHost
```

Depending on if you need the OLED driver you either proceed with

```bash
sudo make -f Makefile.Pi.OLED
sudo make install
```

or just with

```bash
sudo make
sudo make install
```

### Configuring MMDVM.ini to your needs

Now the fun part begins, adapting the configuration file to suit your hardware and make MMDVMHost behave as you want it to.

The configuration file is located in the MMDVMHost folder, but I would recommend saving it directly to

```
/etc/MMDVM.ini
```

which is the standard path, where MMDVMHost looks for it.

After opening you will see, that the file is structured into different sections, each containing various options to choose from.

Luckily the default options usually do not need to be changed. It is mainly your personal info and hardware information which need to be put in.

So copy the file to the default path, open it and make your adjustments in the upcoming sections.

```bash
cp MMDVM.ini /etc/
sudo nano /etc/MMDVM.ini
```

### General settings

```
[General]
Callsign=G9BF
Id=123456
Timeout=180
Duplex=1
# ModeHang=10
RFModeHang=10
NetModeHang=3
Display=None
Daemon=0
```

Set your call sign, your the radio ID and, depending if you have a simplex or duplex hostspot duplex to “0” or 1“. If you have the OLED, set Display to OLED. For testing it's in my option easier to leave daemon at “0” and set it to “1” later, when you set up the systemd-services.

### Info settings

```
[Info]
RXFrequency=435000000
TXFrequency=435000000
Power=1
# The following lines are only needed if a direct connection to a DMR master is being used
Latitude=0.0
Longitude=0.0
Height=0
Location=Nowhere
Description=Multi-Mode Repeater
URL=www.google.co.uk
```

Pretty self-explanitory

### Modem settings

```
[Modem]
# Valid values are "null", "uart", "udp", and (on Linux) "i2c"
Protocol=uart
# The port and speed used for a UART connection
# UARTPort=\\.\COM4
# UARTPort=/dev/ttyACM0
UARTPort=/dev/ttyAMA0
UARTSpeed=115200
# The port and address for an I2C connection
I2CPort=/dev/i2c
I2CAddress=0x22
# IP parameters for UDP connection
ModemAddress=192.168.2.100
ModemPort=3334
LocalAddress=192.168.2.1
LocalPort=3335

TXInvert=1
RXInvert=0
PTTInvert=0
TXDelay=100
RXOffset=0
TXOffset=0
DMRDelay=0
RXLevel=50
TXLevel=50
RXDCOffset=0
TXDCOffset=0
RFLevel=100
# CWIdTXLevel=50
# D-StarTXLevel=50
# DMRTXLevel=50
# YSFTXLevel=50
# P25TXLevel=50
# NXDNTXLevel=50
# M17TXLevel=50
# POCSAGTXLevel=50
# FMTXLevel=50
# AX25TXLevel=50
RSSIMappingFile=RSSI.dat
UseCOSAsLockout=0
Trace=0
Debug=0
```

Here I set the UARTSpeed to 115200

The values for

```
RXOffset
```

and

```
TXOffset
```

vary for every board and should be measured with [MMDVMCal](https://github.com/g4klx/MMDVMCal) on DMR. You can calculate them now or later.

### Excursion: Calibrate your MMDVM HAT

If you don't know your calibrations values and you want to do the calibration immediatelly, get MMDVMCal from source and compile & install it.

```bash
git pull https://github.com/g4klx/MMDVMCal
cd MMDVMCal
sudo make
sudo make install
```

Open it with

```bash
MMDVMCal /dev/ttyAMA0
```

The best instruction on how to do this I found on [K9NPX's homepage](http://www.k9npx.com/2019/02/hotspot-offset-calibration.html), though he explains it with the Pi-Star version of MMDVMCal. But there is no difference in how the program works internally and how you need to calculate your values.

In general you set your hotspot to send a DMR Signal on a specific frequency which you monitor with your DMR handheld.

Then you move the frequency on the hotspot up and down until you just can't receive it anymore. Then you subtract the lower frequency from higher frequency, divide it by two, add the result to the lower frequency and compare this new calculated frequency with the original frequency you sent your signal on. The difference between both frequencies is your offset.

### M17 Settings

To minimize possible errors and complications, which can be caused due to mutual interference, for the first setup turn off all other analog or digital modes except M17.

```
[M17]
Enable=1
CAN=0
SelfOnly=0
TXHang=5
# ModeHang=10
```

```
[M17 Network]
Enable=1
LocalAddress=127.0.0.1
LocalPort=17011
GatewayAddress=127.0.0.1
GatewayPort=17010
# ModeHang=3
Debug=0
```

### OLED settings

If you do have the OLED-display, set your values

```
[OLED]
Type=3
Brightness=0
Invert=0
Scroll=1
Rotate=0
Cast=0
LogoScreensaver=1
```

Again pretty, self-explanatory, except the first question: OLED Type: 3 means 0.96″ and Type 6 means 1.3″.

### Save your settings and test if MMDVMhost starts up

After having modified all needed settings, save everything again with (STRG)+(o).

Now with

```
MMDVMHost
```

the program should start and find your hotspot, but without having M17Gateway yet, M17 does not work yet.

## Install and configure M17Gateway

### Download the M17Gateway source code and compile & install it

So once again it's

```
git clone https://github.com/g4klx/M17Gateway
cd M17Gateway
sudo make
make install
```

### Configuring M17gateway.ini to your needs

The config file M17Gateway.ini should also be copied to the default directory /etc/ to modify it there. The file is less complex them MMDVM.ini, nevertheless a lot of options can be set. As in MMDVM.ini the default settings are mostly good to go and nearly only personal information and preference settings have to be set up.

```
cp M17Gateway.ini /etc/
sudo nano /etc/M17Gateway.ini
```

### General settings

```
[General]
Callsign=G4KLX
# R for Repeaters
Suffix=R
# H for Hotspots
# Suffix=H
RptAddress=127.0.0.1
RptPort=17011
LocalPort=17010
Debug=0
Daemon=0
```

Callsign is self-evident, the suffix should be an H for simplex hotspots. Ports and addresses should be left unchanged, if you do not have special requirements.

Also for testing I would recommend leaving daemon at “0”.

### Info settings

```
[Info]
RXFrequency=430475000
TXFrequency=439475000
Power=1
Latitude=0.0
Longitude=0.0
Height=0
Name=Nowhere
Description=Multi-Mode Repeater
```

RX and TX frequencies should be adapted accordingly.

### Network settings

```
[Network]
Port=17000
HostsFile1=./M17Hosts.txt
HostsFile2=./private/M17Hosts.txt
ReloadTime=60
# Startup=M17-M17_C
Revert=1
HangTime=240
Debug=0
```

By uncommenting #Startup a startup reflector like M17-M17_C can be activated. Until everything works I would also recommend setting

```
Debug=1
```

If you want to be able to change reflectors by a simple console command, set

```
Enable=1
```

in

```
[Remote Commands]
```

### Save your settings and test if M17Gateway starts up

Having done all configuration changes save again with (STRG)+(o). Test with

```
M17Gateway
```

if the program starts up and you configuration file is okay.

Now the time of truth arrives. Open another console and additionally start MMDVMHost, so that in one console M17Gateway is running and M17Gateway in the other.

If debug is on, you should see them talking to each other, exchanging “pings and pong”.

### Setting up Autostart/Systemd

If everything is working as expected, you are ready to setup systemd and make it start automatically on booting.

Now its time to make a deep breath, set

```
Daemon=1
```

in

```
MMDVM.ini
```

and install systemd services.

### Creating mmdvmhost.service

Create a service

```
sudo nano /lib/systemd/system/mmdvmhost.service
```

with the content

```
Description=MMDVMHost Radio Service
After=syslog.target network.target

[Service]
User=root
Type=forking
ExecStart=/usr/local/bin/MMDVMHost
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

and save it. I know root is questionable at least I was not able to get it done otherwise.

Make it executable

```
sudo chmod 755 /lib/systemd/system/mmdvmhost.service
```

and create a symlink

```
sudo ln -s /lib/systemd/system/mmdvmhost.service /etc/systemd/system/mmdvmhost.service
```

### Creating mmdvmhost.timer

Then you need to create a timer, make it executable again and link it.

```bash
sudo nano /lib/systemd/system/mmdvmhost.timer
```

```
[Timer]
OnStartupSec=90

[Install]
WantedBy=multi-user.target
```

```bash
sudo chmod 755 /lib/systemd/system/mmdvmhost.timer
sudo ln -s /lib/systemd/system/mmdvmhost.timer /etc/systemd/system/mmdvmhost.timer
```

### Setting up m17gateway.service

And now only the m17gateway.service is missing:

It's in M17Gateway/systemd, but you can also create it with

```bash
sudo nano /lib/systemd/system/m17gateway.service
```

and copy+paste the content

```
[Unit]
Description=M17Gateway Service
# Description=Place this file in /lib/systemd/system
# Description=KC1AWV 07 SEP 2021
After=network-online.target
Requires=networking.service

[Service]
User=root
#Type=forking
Type=simple
#Group=root
Restart=on-abnormal
RestartSec=3
ExecStart=/usr/local/bin/M17Gateway /etc/M17Gateway.ini
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process

[Install]
WantedBy=multi-user.target
```

I needed to change again the user to root and the type to simple, else in my case it would not work. Then again

```bash
sudo chmod 755 /lib/systemd/system/m17gateway.service
sudo ln -s /lib/systemd/system/m17gateway.service /etc/systemd/system/m17gateway.service
```

Now we need to make systemd learn about the new services, and after a reboot it should load automatically

```bash
sudo systemctl daemon-reload
sudo systemctl enable mmdvmhost.timer
sudo reboot
```

## Upgrading the firmware on an STM32-DVM board

Collated from hotspot_mmdvm and notes from KC1AWV. Tested on Debian 11 with a RepeaterBuilder v4 board.

### Install build tools

```bash
sudo apt install git gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi
```

Install flash tool

```bash
sudo apt install stm32flash
```

Get source

```bash
git clone https://github.com/g4klx/MMDVM
cd MMDVM
git submodule init
git submodule update
```

Build

```bash
make dvm
```

Flash

If using a USB device, edit

```
Makefile
```

Under the

```
deploy-pi
```

section, change

```
/dev/ttyAMA0
```

to the path of your device. eg: `/dev/ttyUSB0`

Close JP1 and apply power

Optionally back-up the existing firmware:

```bash
stm32flash -r /dev/ttyUSB0 stm32-backup.bin
```

```
make deploy-dvm
```
