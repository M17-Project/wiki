# M17 Radio Compatibility List

## Kenwood

| Model            | RX | TX | Comments                |
| ---------------- | -- | -- | ----------------------- |
| TM-D710 (A/G/GA) | ✔️ | ✔️ | TX needs to be inverted |
| TM-V71A          | ✔️ | ✔️ | TX needs to be inverted |

## Motorola

| Model             | RX | TX | Comments |
| ----------------- | -- | -- | -------- | 
| CDM 750/1250/1550 | ✔️ | ✔️ | Configure codeplug to use "flat audio" available on rear accessory connector. |
| DM1400 | ✔️ | ✔️ |  Tested with Module17 - Phase settings: TX: Normal, RX: Normal  |
| GM300             | ✔️ | ✔️ | Demod out is inverted. Adding a 100k resistor is needed for the RX to work. Move jumper JU551 to pos. A. |
| GM340 / GM360 / GM380 | ✔️ | ✔️ | Phase settings: TX: Inverted, RX: Inverted. Tested with Module17 - Reported by OE1JTB & OE3TEC |
| GM900             | ❓ | ❓ |            |
| GM1200            | ✔️ | ✔️ | No HW mod necessary for RX. |

## Yaesu

| Model   | RX | TX | Comments |
| ------- | -- | -- | -------- |
| FT-817  | ✔️ | ✔️ | Tested with [STM32-DVM-USB](http://www.repeater-builder.com/products/stm32-dvm.html) and latest [MMDVM Firmware](https://github.com/g4klx/MMDVM) using [M17Client](https://github.com/g4klx/M17Client). No modification needed. |
| FT-818  | ✔️ | ✔️ | |
| FT-857(D) | ✔️ | ✔️ | Tested with Module17. Phase settings: TX: Inverted, RX: Inverted |
| FT-897 | ✔️ | ✔️ | Tested with Module17. Phase settings: TX: Inverted, RX: Inverted |
| FT-991A | ✔️ | ✔️ | Tested with Module17 |
| FTM-100  | ✔️ | ✔️ | (reported by G4KLX)|
| FTM-200  | ✔️ | ✔️ | Reported by KD2YFY|
| FTM-300  | ✔️ | ✔️ | (reported by G4KLX) Phase settings: 70cm: TX: Normal, RX: Normal, 2m: TX: Inverted, RX: Inverted |
| FTM-400  | ✔️ | ✔️ | Tested with Module17 (by ON4BCY) Phase settings: 70cm: TX: Normal, RX: Normal, 2m: TX: Inverted, RX: Inverted|
| FTM-500DE  | ✔️ | ✔️ | Tested with Module17. Phase settings: TX: Normal, RX: Inverted (Only on 70cm) |
| FT-7800 | ✔️ | ✔️ | |
| FT-7900R | ✔️ | ✔️ | (reported by G4KLX)|
| FT-8000 | ✔️ | ❓ | No modification needed. Use the 9600baud output at the TNC connector. |
| FT-8900 | ✔️ | ✔️ | Phase settings: TX: Normal, RX: Inverted (Only on 70cm) Reported by OE3TEC |
| FTM-6000 | ✔️ | ✔️ | No modification needed. Use the 9600baud output at the TNC connector. |

## Icom

| Model       | RX | TX | Comments |
| ----------- | -- | -- | -------- |
| IC-706mkIIg | ✔️ | ✔️ | |
| ID-880H     | ✔️ | ✔️ | inverted TX and RX (reported by W9RNR) |
| IC-5060/6060| ✔️ | ✔️ | inverted TX. Supports the use of the internal speaker/microphone. See [this section](icf5060_3060.md) |
| IC-3060/4060| ✔️ | ✔️ | inverted TX. Supports the use of the internal speaker/microphone. See [this section](icf5060_3060.md) |
| IC-e2820    | ✔️ | ✔️ | inverted TX and RX (reported by ON4MOD) |
| IC-9100     | ✔️ | ✔️ | inverted TX and RX (reported by ON4MOD) |

## TYT

| Model        | RX | TX | Comments |
| ------------ | -- | -- | -------- |
| MD-380/390   | ✔️ | ✔️ | Both RX and TX work with [OpenRTX](https://openrtx.org) custom firmware.  |
| MD-UV380/390 | ✔️ | ✔️ | Both RX and TX work with [OpenRTX](https://openrtx.org) custom firmware.  |

## Retevis

| Model    | RX | TX | Comments |
| -------- | -- | -- | -------- |
| RT3/RT3S | ✔️ | ✔️ | Both RX and TX work with [OpenRTX](https://openrtx.org) custom firmware. |
