<!-- Add links to top -->

Setup Guide <!-- omit in toc -->
===============================================
<span style=""><img src="images/moode-r900-logotype-clear-black-cropped.png"
width="300px"></span>

(C) Tim Curtis 2017  
(C) @azimuth 2024 Pi Imager tutorial

### Table of Contents <!-- omit in toc -->

- [1. Introduction](#1-introduction)
- [2. Creating the Boot SD Card](#2-creating-the-boot-sd-card)
- [3. After Startup](#3-after-startup)
  - [3.1. Audio Device](#31-audio-device)
  - [3.2. Music Files](#32-music-files)
  - [3.3. Hotspot](#33-hotspot)
  - [3.4. File Sharing](#34-file-sharing)
  - [3.5. Software Updates](#35-software-updates)
- [4. Imager Tutorial](#4-imager-tutorial)
- [5. Advanced](#5-advanced)
  - [5.1. REST API](#51-rest-api)
  - [5.2. SSH Commands](#52-ssh-commands)

# 1. Introduction

Moode audio player is a Free Open Source Software (FOSS) music streamer for the wonderful Raspberry Pi family of Single Board Computers (SBC). To access the player enter http:// moode, moode.local or ip_address in a Web Browser. If you are using a Smartphone use the "Save to Home Screen" option on IOS or Android to create a Moode App.

[Quick help](https://github.com/moode-player/docs/blob/main/Quickhelp.pdf)  contains instructions for navigating Moode and using its features including Multiroom audio. Quick help is located on the "m" menu in the player.

# 2. Creating the boot SD Card

Before you can access Moode a pre-built image must be written to a micro SD Card using the official Raspberry Pi Imager. Download links for the Imager and Moode pre-built images are available on the Downloads page at [moodeaudio.org](https://moodeaudio.org) Pre-built images can also be conveniently selected directly in the Media Player OS section of the Imager. Refer to the included [Tutorial](#4-imager-tutorial) for step by step instructions.

Note: For improved security pre-built images do not contain a default logon userid, WiFi SSID or Hotspot password and Secure Shell (SSH) access is disabled by default. These are setup using the Imager.

# 3. After Startup

After Moode starts up and you access it in your Web Browser you can configure your audio device, add music files to the Library and make use of features that enhance the usability and function of Moode.

## 3.1. Audio Device

## 3.2. Music Files

## 3.3. Hotspot

Moode provides a WiFi Hotspot on the 2.4 GHz WiFi band. Refer to the
default settings below.

- SSID          Moode
- Password      As set in Network Config screen
- URL           http:// moode.local or 172.24.1.1

The Hotspot starts when any of the following are true.

- WiFi SSID is set to "Activate Hotspot" in Network Config.
- WiFi SSID is defined in Network Config but no IP address was assigned after
attempting to connect to the configured SSID or any saved SSID's.

## 3.4. File Sharing

Moode can be used as a File Server by turning on one or more of the file sharing protocols listed below. These protocols and their respective options are in the File Sharing section of System Config.

- SMB File Sharing. SMB (Samba) shares named NAS, Playlists, and SDCard are automatically created. Each USB disk will also have a Samba share created that is named after its Disk Label.

- NFS File Sharing. Access and options defaults are provided but can be manually overridden. Each USB disk will have an NFS export created whose path is /media/disk_label.

- DLNA File Sharing. The DLNA media server indexes music sources defined for MPD and makes them available to UPnP control points.

## 3.5. In-Place Software Updates

Updates to Moode are available via "CHECK for software update" in System Config. The updates are keyed to a major release series for example Moode 9. There are no in-place updates across major releases for example Moode 8 to Moode 9. Each new major release requires a fresh image.

# 4. Imager Tutorial

Moode requires using the official Raspberry Pi Imager to configure the OS image before writing it to an SD Card. Follow these step by step instructions to carry out this process.

1. [Download](https://www.raspberrypi.com/software/) the official Raspberry Pi Imager.
<div style="margin-bottom:1em"><img src="images/imager_1.png" width="400px"></div>

2. Click CHOOSE OS and scroll down to "Media Player OS" and then to "moOde audio player" and then click the version of moOde you would like to install.
<div style="margin-bottom:1em"><img src="images/imager_2.png" width="400px"></div>
<div style="margin-bottom:1em"><img src="images/imager_3.png" width="400px"></div>
<div style="margin-bottom:1em"><img src="images/imager_4.png" width="400px"></div>

  Or if you have downloaded moOde directly from the Download page at [moodeaudio.org](https://moodeaudio.org), then scroll down to “Use custom” and select the downloaded zip file from the file chooser.
<div style="margin-bottom:1em"><img src="images/imager_5.png" width="400px"></div>

3. After the OS is selected click CHOOSE STORAGE then click the SD card drive where the OS image will be written.
<div style="margin-bottom:1em"><img src="images/imager_6.png" width="400px"></div>

4. Click NEXT to display the OS customization screen
<div style="margin-bottom:1em"><img src="images/imager_7.png" width="400px"></div>
<div style="margin-bottom:1em"><img src="images/imager_8.png" width="400px"></div>

5. Click EDIT SETTINGS to display the OS settings screen. On the GENERAL tab enter your desired hostname, userid and password, optional WiFi SSID and password and finally the Locale.
<div style="margin-bottom:1em"><img src="images/imager_9.png" width="400px"></div>

6. Click the SERVICES tab and enable SSH with password authentication.
<div style="margin-bottom:1em"><img src="images/imager_10.png" width="400px"></div>

7. Click SAVE which will return you to the OS Customization screen.
<div style="margin-bottom:1em"><img src="images/imager_11.png" width="400px"></div>

8. Click YES to apply the customizations and display the confirmation screen.
<div style="margin-bottom:1em"><img src="images/imager_12.png" width="400px"></div>

9. Click YES to proceed and write the OS image to the SD Card and automatically verify it after the write completes.
<div style="margin-bottom:1em"><img src="images/imager_13.png" width="400px"></div>
<div style="margin-bottom:1em"><img src="images/imager_14.png" width="400px"></div>

10. After verification is complete the SD Card can be removed from the drive and inserted into a Raspberry Pi for initial startup.
<div style="margin-bottom:1em"><img src="images/imager_15.png" width="400px"></div>

# 5. Advanced

## 5.1. REST API

If an HTTP command returns data it is in JSON format following REST guidelines.
The base URL is http://moode/command/?cmd=

get_currentsong
Returns contents of the file /var/local/www/currentsong.txt.
Turn on the Metadata file option in Audio Config to generate this file.

get_output_format
ALSA output format or 'Not playing' is returned.

get_volume
Returns the Knob volume.

set_volume
Sets the knob volume to value N, up or down N or mute toggle.
Arguments: N | -up N | -dn N | -mute

get_cdsp_config
Returns the current CamillaDSP config name

set_cdsp_config
Sets CamillaDSP to the specified config name.
Arguments: A config name from the list of available configs including 'Off'.

set_coverview
Turns CoverView screen saver on or off.
Arguments: -on | -off

upd_library
Submits an "Update library" command.

restart_renderer
Restarts the specified renderer.
Arguments: --bluetooth | --airplay | --spotify | --squeezelite | --plexamp | --roonbridge

MPD commands
See MPD protocol for list of commands.
https://mpd.readthedocs.io/en/latest/protocol.html

Deprecated REST API (http) commands. The following commands have been replaced
by the equivalent new commands above and at some point will not be supported. It is recommened to update your scripts to use the new commands.

- vol.sh
- coverview.php
- libupd-submit.php
- restart-renderer.php

## 5.2. SSH Commands

moodeutl
This command can be used for printing logs, status or for manipulating certain
parts of moOde. For a list of options type moodeutl --help

mpc
This command can be used to control MPD. For a list of options type mpc help

vol.sh
This command can be used to get or set MPD volume and update the volume knob.
For a list of options type /var/www/util/vol.sh --help
To run it type /var/www/util/vol.sh <options>

libupd-submit.php
This command submits a "Library update".
To run it type /var/www/util/libupd-submit.php

coverview.php
This command turns the CoverView screen saver on or off.
To run it type /var/www/util/coverview.php -on | -off

restart-renderer.php
This command restarts the specified renderer.
To run it type /var/www/util/restart-renderer.php --renderer
--bluetooth    Restart Bluetooth
--airplay      Restart AirPlay
--spotify      Restart Spotify Connect
--squeezelite  Restart Squeezelite
--plexamp      Restart Plexamp
--roonbridge   Restart RoonBridge


---------------Old stuff---------------
8. Player Setup And Configuration

1. INITIAL SETUP

 a) Insert boot SD card
 b) Insert ethernet cable or alternatively use WiFi SSID defined in Pi Imager
 b) Power on
 c) http://moode | http://moode.local | http://IP_ADDRESS

9. Audio Device Setup

 - USB DEVICE
 a) Plug in USB audio device
 a) Menu, Configure, Audio
 c) Set Output device to to the name of the USB audio device

 - I2S DEVICE
 a) Menu, Configure, Audio
 b) Set Named I2S device or DT overlay to the correct device or overlay name
 c) Menu, Power, Restart
 d) Menu, Configure, Audio
 c) Set Output device to to the name of the I2S audio device

10. Add Music Files

 - USB STORAGE DEVICES
 a) Insert USB storage device
 b) Menu, Update library
 c) Wait for completion (no spinner)

 - BOOT SDCARD STORAGE
 a) Menu, Update library
 b) Wait for completion (no spinner)

 - NAS DEVICE
 a) Menu, Configure, Library
 b) CREATE Music source
 c) After SAVE, return to Playback or Library
 d) Menu, Update library
 e) Wait for completion (no spinner)

5. VERIFY AUDIO PLAYBACK

 a) http://moode | http://moode.local | http://IP_ADDRESS
 b) Play one of the radio stations
 c) Switch to Library Folder view
 d) Navigate to the SDCARD/Stereo Test
 e) Play the "LR Channel And Phase" track

11. Custom Configurations

Customize the player by using any of the following procedures.

1. CONFIGURE FOR WIFI-ONLY CONNECTION

 - Ethernet cable connected
 a) Insert WiFi adapter or use Pi Integrated WiFi
 b) http://moode | http://moode.local | http://IP_ADDRESS
 c) Menu, Configure, Network
 d) Configure a WiFi connection
 e) Menu, Power, Shutdown
 f) Unplug Ethernet cable
 g) Power on

 - Hotspot mode
 a) Join Hotspot SSID, password = Refer to MOODE OS IMAGE section
 b) http://moode.local | http://172.24.1.1
 c) Menu, Configure, Network
 d) Configure a WiFi connection
 e) Menu, Power, Restart

2. SWITCH FROM WIFI-ONLY BACK TO ETHERNET-ONLY

    a) Plug in Ethernet cable
    b) Menu, Configure, Network
    c) RESET network configuration to defaults
    d) Menu, Power, Shutdown
    e) Remove WiFi adapter
    f) Power on
