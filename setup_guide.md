<!-- Add links to top -->

Setup Guide <!-- omit in toc -->
===============================================
<span style="float:right;margin-right:1em;margin-top:-5em"><img src="../images/moode-r900-logotype-clear-black-cropped.png"
width="200px"></span>
(C) Tim Curtis 2017  
(C) @azimuth 2024 Pi Imager section

### Table of Contents <!-- omit in toc -->

- [1. Introduction](#1-introduction)
- [2. Creating the Boot SD Card](#2-creating-the-boot-sd-card)
- [3. After Startup](#3-after-startup)
  - [3.1. Audio Device](#31-audio-device)
  - [3.2. Music Files](#32-music-files)
  - [3.3. Networking](#33-networking)
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

Before you can access Moode a pre-built image must be written to a micro SD Card using the official Raspberry Pi Imager. Download links for the Imager and Moode pre-built images are available on the Downloads page at [moodeaudio.org](https://moodeaudio.org) Pre-built images can also be conveniently selected directly in the Media Player OS section of the Imager. Refer to this [Tutorial](#N-imager-tutorial) for step by step instructions.

Note: For improved security pre-built images do not contain a default logon userid, WiFi SSID or Hotspot password and Secure Shell (SSH) access is disabled by default. These are setup using the Imager.

# 3. After Startup

After Moode starts up and you access it in your Web Browser you can configure your audio device, add music files to the Library and make use of several  features that enhance the usability and function of Moode.

## 3.1. Audio Device

## 3.2. Music Files

## 3.3. Networking

Moode provides a WiFi Hotspot on the 2.4 GHz WiFi band. Refer to the
default settings below.

- SSID          Moode
- Password      As set in Network Config screen
- URL           http:// moode.local or 172.24.1.1

The Hotspot starts automatically when any of the following are true.

- WiFi SSID is set to "Activate Hotspot" in Network Config.
- WiFi SSID is defined in Network Config but no IP address was assigned after
attempting to connect to the configured SSID or any saved SSID's.

## 3.4. File Sharing

Moode can be used as a File Server by turning on one or more of the file sharing protocols listed in System Config.

SMB File Sharing. SMB (Samba) shares named NAS, Playlists, and SDCard are automatically created. Each USB disk will also have a Samba share created that is named after its Disk Label.

NFS File Sharing. Access and options defaults are provided but can be manually overridden. Each USB disk will have an NFS export created whose path is /media/disk_label.

DLNA File Sharing.  
description goes here.

## 3.5. In-Place Software Updates

Updates to Moode are made available periodically and are downloaded
and installed by clicking "CHECK for software update" in System Config.

# 4. Imager Tutorial

Text and images go here

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
