# Twilio Wireless Broadband IoT Developer Kit

Twilio's Wireless Broadband IoT Developer Kit is a collection of hardware and pre-configured software which enables rapid prototyping on Raspberry Pi.

This repository contains a parts manifest, documentation, a pre-built custom Raspbian SD card image, and the scripts we used to generate the custom Raspbian image.  The scripts are provided for your reference only and are not required to get up and running with the Broadband IoT Developer Kit.

## Contents

1. [Hardware](#hardware)
1. [Software](#software)
1. [Connections](#connections)
1. [Installation Options](#installation_options)
1. [Getting Started](#getting_started)

## Hardware

- [Raspberry PI 3 B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) In addition to the processor upgrades the 3 B+ gives, due to the use of USB along with the Seeed HAT below, we recommend it specifically due to its built-in protections around back-powering and improved support for hot-plugging USB peripherals.
- [Seeed LTE Cat 1 Pi HAT USA-AT&T or Europe version](http://wiki.seeedstudio.com/LTE_Cat_1_Pi_HAT/), depending on your region.
- [A micro-usb cable](https://www.amazon.com/gp/product/B01FA4JXN0) for connecting the HAT to the Pi (more on why we need this later)
- A suitable power supply for the PI 3B+.  We strongly recommend using a known compatible adapter, such as the [CanaKit 2.5A Raspberry Pi 3 B+ Power Supply](https://www.amazon.com/dp/B07GZZT7DN)
- Compatible I2C Grove sensors if you wish to work with the on-board Grove headers on the HAT.
- [Micro SD card suitable for Raspberry Pi](https://www.raspberrypi.org/documentation/installation/sd-cards.md) - we like to use at least 8GB Class 10 cards for speed and extra space.

Additionally, you will find handy access to a USB keyboard and HDMI monitor (with cable) for initial setup.  After setup, our build can be run completely headless.  If you choose to purchase a case, ensure it is one you can fit a Raspberry Pi HAT into.  Be aware that the I2C grove headers on the Seeed HAT mentioned above protrude a bit taller than some other HAT's and may require additional room.

## Software

- Raspbian Stretch Lite
- Twilio Trust Onboard SDK
- Azure IoT SDK for C and Python
- smstools
- ppp

The base Broadband IoT image contains the following setup out of the box:

- Raspbian Stretch Lite distribution configured to support the Seeed LTE HAT and its I2C Grove headers
- PPP daemon configured to connect using an attached Twilio LTE SIM for internet connectivity
- A DHCP daemon to vend IP addresses when connected directly via Ethernet for easy console connectivity
- Trust Onboard SDK for interfacing with X.509 capabilities of Twilio Trust Onboard SIMs
- Microsoft Azure IoT SDKs for Python and C compiled and configured
- smstools and procmail configured to deliver inbound SMS messages to a shell script for routing and action

# Connections

1. Put the Twilio ToB SIM into the Seeed LTE Cat 1 HAT
1. Attach the provided LTE antennas to the Seeed LTE Cat 1 HAT
1. Attach the HAT to the Raspberry Pi 3 B+
1. Plug a USB A -> USB Micro B cable from one of the USB ports on the Raspberry Pi into the micro USB port on the HAT
1. Insert your Raspbian micro SD card
1. Attach a monitor by HDMI and keyboard for initial setup
1. Plug in your Raspberry Pi power supply

A quick aside - if you have worked with Raspberry Pi HAT's before, you may have wondered why we will need a separate USB cable to attach to the Pi to the HAT in addition to using the Pi's expansion header.  The reason for this is while the Raspberry Pi expansion header exposes a single UART to the Seeed LTE Cat 1 HAT, the u-blox LARA-R2 cellular module the HAT integrates is capable of exposing 6 discrete UARTs and does so when connected by USB.  Three of these UART's can be used for AT and data simultaneously without having to configure and support multiplexing or having to suspend operations such as a running PPP connection in order to run other AT commands.  If your usecase does not require simultaneous access to ToB and PPP, you can safely skip the USB connection but for purposes of getting started, we recommend using the USB connection.

## Installation Options

We have a Raspberry Pi image, based on Raspbian (Stretch) available which includes a fully configured environment, the Breakout Trust Onboard C SDK, and the Azure IoT for C SDK set up as well as ppp and SMS connectivity.  You can [download this image here](https://github.com/twilio/Wireless_Broadband_IoT_Dev_Kit/releases).

_We have included the required steps to [replicate the environment yourself](image_builder/README.md).  This is provided for informational purposes only, we encourage you to use the provided pre-built SD card images for the optimal experience with the kit._

After downloading the SD card image, you can unpack the archive and use [Etcher](https://etcher.io/) (or the tool/method of your choice) to write the image out to the SD card.

The default login is the same as Raspbian: `pi` and `things`.

## Connectivity

By default, the Broadband IoT Developer Kit has the following connectivity enabled:

- You can directly connect an HDMI monitor and USB keyboard to the Pi to obtain console if desired.  The HDMI monitor should be plugged in at power-on time for it to be detected and enabled.
- `ppp` is configured to auto-start and connect to Twilio Wireless
- `udhcpd` is configured to auto-start on the Pi's build in ethernet port:
  - A static ip address of `192.168.253.100` will be assigned to the Pi
  - The Pi will assign DHCP address to clients over ethernet in the range of `192.168.253.101` - `192.168.253.199`
- `ssh` is configured and accessible via the default static ip address
- `smstools` is configured to listen for incoming SMS to your SIM.  These messages by default are routed to a bash script in `/home/pi/sms_received.sh`.  This can be changed in the configuration file `/etc/smsd.conf`
- By default, WiFi is not enabled.  WiFi can be configured using `raspi-config` after connecting to the Pi.  Please be sure to change the default password before doing this for security.

**IMPORTANT:** Some differences in our developer kit image from stock Raspbian are important to note:

- A DHCP server will be running on the Pi's ethernet port by default.  This enables headless connectivity out of the box but will cause problems if the Pi is conncected by ethernet to a network with another DHCP server.  If you are connecting the Pi's ethernet to an existing network, you should first disable `udhcpd` with `sudo systemctl disable udhcpd`.  You can then disable the static ip assignment in `/etc/dhcpd.conf` by commenting out `interface eth0` and `static ip_address=192.168.253.100/24`
- `sshd` is enabled by default and the `pi` account has a documented default password.  At minimum the password should be changed immediately and your needs around leaving `sshd` enabled should be evaluated.  To disable `sshd`, you can run `sudo systemctl disable ssh`.
- `ppp` will start up and connect to the internet using the installed SIM card.  This will result in billable activity so performing actions like updating the system packages should be done with care.  To disable auto-start of `ppp` on boot, you can: `sudo systemctl disable twilio_ppp`
- `smstools` is enabled by default, you can disable it with `sudo systemctl disable sms`
