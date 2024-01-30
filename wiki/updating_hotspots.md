# Updating Hotspots to add M17 support

This guide should help you set up your hotspot to work with M17 mode.

Windows:- flash the microsd card the [WPSD disk image](https://w0chp.net/WPSD_Latest.img.xz); we recommend [BalenaEtcher](https://www.balena.io/etcher) or [Raspberry Pi Imager](https://www.raspberrypi.com/software/); as they will automatically decompress and flash the image for you.
  - Two ways to setup wireless:
    1. use the [WPSD Wifi Config File Generator](https://w0chp.net/wpa-config-generator/), and place the downloaded WPA config file into the `/boot` volume of the SD card.
        - then place the sdcard into the hotspot and power it up.
    2. boot up the device with the microsd card in, then wait for it to start its WiFi access point,
        - find out what the IP address of the device is, try using "Advanced IP Scanner"*,
        - use PuTTY to connect to the raspberry pi over ssh; the default login is username: `pi-star` password: `raspberry`,
        - run `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`,
        - add the following lines:<br>

        ```
        network={
               ssid="YourWifi"
               psk="YourPassword"
        }
        ```

        - press Ctrl-O to overwrite, then Enter to confirm, finally Ctrl-X to exit the editor,
        - reboot the device with `sudo reboot`,
- wait for the hostpot to login to your access point, then use its IP address to access the configuration webpage,
- go to Admin -> Update; let the process finish (the default login is username: `pi-star` password: `raspberry`),
- [make a note of which modem you have and its associated command from this list](https://w0chp.net/musings/update-modem-fw-wpsd/), so that you can update your firmware in the steps:
- use `ssh` again to access raspberry pi,
- run the following command, based on your modem type from the list (this uses a ZUMspot as an example):<br>
- `sudo sudo pistar-zumspotflash rpi`
- done

*Linux users can use `nmap -sP ...` instead

Your hotspot is ready to roll. Under Linux, use `ssh` in the terminal instead of PuTTY and `dd` to flash the image ([BalenaEtcher](https://www.balena.io/etcher) or [Raspberry Pi Imager](https://www.raspberrypi.com/software/) are fine too!). You might also want to set the correct display settings (if your board has one). For the Zumspot RPi HAT, I had to select 115200 baud /dev/ttyAMA0 UART and "OLED type-6". Don't forget to enable M17 in the dashboard (and select an appropriate reflector).

Big thanks to Chip `W0CHP` for his marvelous work.

References:
1. [Chip's WPSD website](https://wpsd.radio/)
2. [Updating MMDVM, by Michael Clemens, DK1MI (ex DL6MHC)](https://qrz.is/m17-md380-openrtx-pistar-mmdvm/)
