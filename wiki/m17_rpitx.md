# Transmitting M17 with rpitx

## Introduction

rpitx is an amazing piece of software by F5OEO that allows you to experiment with transmitting RF using only a Raspberry Pi. rpitx includes many different modes, such as:

* FM Carrier (Transmit an unmodulated FM carrier)
* Chirp (Sweep a signal through a range of frequencies)
* Spectrum (display a picture in the waterfall)
* RfMyFace (take a picture from RPi camera, display on waterfall)
* FM with RDS (transmit stereo wideband FM with RDS information)
* SSB (transmit single-sideband audio)
* SSTV (transmit a photo using SSTV)
* POCSAG (transmit pager tones)
* FreeDV (transmit voice and data using an open-source codec)
* Opera (transmit an Opera beacon)

This document will help you set up a Raspberry Pi for rpitx, convert a sound file to M17 baseband, and then transmit it from the Raspberry Pi.

## What You Need

You will need the following:

* Raspberry Pi with SSH access - I am using a Raspberry Pi 3B+
  * Preferably a Raspberry Pi OS Lite install, there's no need for a full desktop
* (Optional) A short piece of wire to act as an antenna
* (Optional) A RTL-SDR or other SDR dongle to receive your signal

That's pretty much it!

## Instructions

### Build the Tools

First, log into your Raspberry Pi via SSH. We will be doing all this work via SSH, due to a few users stating that the HDMI output has a tendency to disappear when transmitting. I have not experienced this myself, due to only working on a headless RPi, but all of the programs are run from the command line anyway, so there's no need for a full Raspberry Pi OS desktop install.

Next, we need to install some tools. We'll start with codec2.

#### Set Up Your Environment

Get a few tools for getting source code and compiling

```bash
sudo apt install git build-essential cmake
```

Install sox - will need this later for generating baseband audio

```bash
sudo apt install sox
```

Create a working folder for source code

```bash
mkdir git && cd git
```

#### Compile and Install codec2

```bash
git clone https://github.com/drowe67/codec2.git
cd codec2
mkdir build_linux && cd build_linux
cmake ..
make
sudo make install
cd ../..
sudo ldconfig
```

#### Compile and Install m17-cxx-demod

```bash
sudo apt install libboost-program-options1.67-dev libgtest-dev
git clone https://github.com/mobilinkd/m17-cxx-demod.git
cd m17-cxx-demod
mkdir build && cd build
cmake ..
make
make test
sudo make install
cd ../..
```

#### Compile and Install rpitx

This install will take a while, it pulls in many dependencies and self-installs. Get yourself a cup of coffee while you wait.

```bash
git clone https://github.com/F5OEO/rpitx
cd rpitx
./install.sh
```

When finished compiling, you will see this prompt:

```
In order to run properly, rpitx need to modify /boot/config.txt. Are you sure (y/n)
```

Be sure to accept (press Y and Enter) in order for the proper boot options to be set. Then, finally reboot the Raspberry Pi

```bash
sudo reboot
```

### Optional Addition

Jumper wires with female Dupont connectors Cut a wire to a length that's appropriate for the frequency you wish to transmit on. rpitx reportedly can transmit on frequencies from 5 KHz up to 1500 MHz. For example, I used a spare jumper wire with female Dupont connectors on it. As I hold a license for ham radio, I cut the wire for the 70cm band, about 150cm-160cm long. If you do not have a ham radio license, cut the wire for an acceptable ISM band for your location.

#### CAVEAT

While the Raspberry Pi may not have a lot of transmitting power using rpitx, I strongly suggest staying off any licensed land mobile or broadcast frequencies. Better safe then sorry.

## The FUN Stuff!

### Generate M17 Baseband

Now that we have all the tools and programs installed, it's time to generate a file that contains M17 baseband audio. We will use m17-mod for this.

From the m17-cxx-demod GitHub README:

> **m17-cxx-mod**
> —
> This program reads in an 8k sample per second, 16-bit, 1 channel raw audio stream from STDIN and writes out an M17 4-FSK baseband stream at 48k SPS, 16-bit, 1 channel to STDOUT.

Using this information, you can either create your own audio file or use an existing audio file that fits the requirements. I use Audacity to create files for my use. You could also download a speech sample file from the Open Speech Repository.

Let's create a working directory to use for playing with M17 stuff. 

```bash
cd
mkdir m17 && cd m17
```

Place your audio file in the working directory. We will now use m17-mod in order to generate the M17 baseband audio.

```bash
sox <filename> -t raw - | m17-mod -S <callsign> > m17baseband.wav
```

 Replace <filename> with the name of the original .wav audio file and <callsign> with your ham radio callsign. If you do not have a ham radio license, you can use something like TEST for the callsign instead.

The command as entered does the following:

* Play the original wav file as raw to STDOUT
* Pipe the STDOUT from sox into m17-mod
* m17-mod attaches the callsign to the M17 metadata
* Output the generated M17 baseband to a new .wav file

The process of generating the baseband audio may take a few seconds to a minute or two, depending on the length of the original audio file.

### Transmit M17 Baseband

Great, you should now have a new file with the generated baseband audio. Now, it's time to send that through rpitx. Run the following command:

```bash
cat m17baseband.wav | csdr convert_i16_f | csdr gain_ff 4200 | csdr convert_f_samplerf 20833 | sudo rpitx -i- -m RF -f <frequency>
```

Change <frequency> to the frequency you want to transmit at. For example, I chose a frequency in the 70cm ham band of 438.850MHz, which would look like -f 438.850e3 in the command above.

**FURTHER EXPLANATION OF COMMANDS NEEDED HERE**

This command will take the baseband audio that you generated, do some DSP magic on it, and then transmit it all through GPIO 4 (Pin 7) on the Raspberry Pi.

Congratulations, you've transmitted M17 using only a Raspberry Pi!

## OPTIONAL

Use an RTL-SDR dongle or similar SDR receiver to listen to the baseband audio, or pipe the received baseband into m17-demod to diagnose, test, and listen to your transmission!

[![M17 with rpitx on OpenWebRX](https://i.ytimg.com/an_webp/NvrPTQ9R39E/mqdefault_6s.webp?du=3000&sqp=CMTZ-6EG&rs=AOn4CLAPpKis1Q2cE3WQHG4_NUooIxiOWw)](https://www.youtube.com/watch?v=NvrPTQ9R39E "KC1AWV and rpitx")