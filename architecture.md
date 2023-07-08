Architecture Document <!-- omit in toc -->
==========================================
Updated: 2023-07-8

This document provides information on the software architecture, with source references as well as
some guidance into how to extend/change the code.

### Table of Contents <!-- omit in toc -->

- [1. Overview](#1-overview)
- [2. Backends](#2-backends)
  - [2.1 Main backend](#21-main-backend)
    - [2.1.1 Interface MPD](#211-interface-mpd)
  - [2.2 Multiroom backend](#22-multiroom-backend)
    - [2.2.1 Interface Multiroom clients](#221-interface-multiroom-clients)
    - [2.2.2 Interface client to backend](#222-interface-client-to-backend)
  - [2.3 Playback modes](#23-playback-modes)
  - [2.4 Interface web client](#24-interface-web-client)
- [3. Web client](#1-web-client)
  - [3.1 Interface to backend](#31-interface-to-backend)
  - [3.2 Page layout](#32-page-layout)
- [4. General comments](#4-general-comments)

# 1. Overview

TBD.

# 2. Backends

TBD.

## 2.1 Main backend

TBD.

### 2.1.1 Interface MPD

TBD.

## 2.2 Multiroom backend

TBD.

### 2.2.1 Interface Multiroom clients

TBD.

### 2.2.2 Interface client to backend

TBD.

### 2.3 Playback modes

TBD.

### 2.4 Interface web client

TBD.

# 3. Web client

TBD.

## 3.1 Interface to backend

TBD.

# 3.2 Page layout

# 4. General comments








The application architecture is summarized in the Overview section of the Master Development Guide. It's based on the original 2013 tsunamp player written by Andrea Coiutti & Simone De Gregori from which moOde and other similar projects were forked. https://github.com/moode-player/docs/blo...20guide.md

There are no details in the document describing how all the parts of application are stitched together due to lack of time to write that section. Similarly the Source Tree documentation which would be related to that section is incomplete and outdated. Volunteers are welcome :-)

I can get you started with the following quick run through.

Each web page consists of:

1. /var/www/header.php
2. An HTML template file from /var/www/templates/
3. /var/www/footer.php

The main Playback/Library screen is /var/www/templates/indextpl.html and /var/www/index.php. The rest of the files in /var/www/templates/ are the Configtration screens with corresponding PHP scripts in /var/www/

When a web page loads into the Browser the jQuery(document).ready code is executed in /var/www/js/scripts-panels.js or scripts-configs.js depending on whether the page is indextpl.html or one of the Configuration HTML templates respectively.

In these two JS scripts, settings and other data is fetched from SQL tables on the back-end and then AJAX connections to the two "server engines" are made in the section below via functions in /var/www/js/playerlib.js

Code:
// Connect to server engines
engineMpd();
engineCmd();

The MPD engine is /var/www/engine-mpd.php. It's a PHP wrapper around the MPD idle timeout loop which waits for MPD state change events and emits the state data back to the front-end JS engineMpd() function in playerlib.js. This function establishes permanent AJAX while loop with engine-mpd.php.

The Command engine is /var/www/engine-cmd.php. It's based on straight sockets and serves as a way for PHP scripts to send commands back to the front-end via the sendEngCmd() function in /var/www/inc/common.php. The commands are processed by the front-end JS engineCmd() function in playerlib.js. This function establishes a permanent AJAX while loop with engine-cmd.php.

The other core part of the application is /var/www/daemon/worker.php. Its function is to perform startup, process jobs sent to it by the front-end and handle player state when any of the "Renderers" connect/disconnect. Look in the section below where this polling loop is started.

(1) Turn on Debug logging in System Config to see in the Browser console whats returned by engine-mpd.php to the front-end, and in /var/log/moode.log to see some of the activity in worker.php and other PHP functions.
(2) The Library JSON files are cached in /var/local/www/

Code:
// BEGIN WORKER EVENT LOOP

A couple other application things to take note of:

1. Although the application is complex with many features, the default audio pipeline is simple and has not changed from the original over 10 years ago. It's still MPD -> ALSA -> Device.

2. The WebUI is designed to scale and adapt to screen sizes including Smartphone, Tablet, Desktop, TV and various locally  attached screens for example Pi 7" Touch or Ultra-wide LCD/OLED displays.

3. The data used to populate Library Tag and Album views is extracted from the MPD database by function loadLibrary($sock) in /var/www/inc/music-library.php and sent in its entirety as a JSON file to the front-end function loadLibrary() in /var/www/js/scripts-library.js which renders it.

4. The application is designed and optimized for the Raspberry Pi family of single board computers. No other platforms are supported.

5. Music Services are not integrated because their API's are proprietary (not Open Source) and they require annual license fees with contracts and NDA's.

6. Development is done using Gulp https://github.com/moode-player/docs/blo...th_gulp.md

7. The release images are built using pi-gen (the tool used to create Raspberry Pi OS images) and standard Debian packages. See our pkgbuild and imgbuild repos. The Raspberry Pi OS packages that are used are current as of the date our release images are generated.

8. We generally stay in sync with the latest official Raspberry Pi OS and kernel releases and 3rd party component releases for example shairport-sync, MPD, librespot, etc.

-------
------


Playback Modes:

1. MPD
This is the core playback application.

- Play song files and Internet radio stations to a locally attached audio device.
- Concurrently but not synchronized, encode the currently playing file or station as an HTTP stream on default port 8000

2. Audio Receivers
These are 3rd party renderer (receive-only) applications integrated into moOde that receive streams sent by client applications using the protocols below.

- Bluetooth
- Airplay 1 and 2
- Spotify Connect
- SlimProto (Squeezelite)
- RAAT (RoonBridge)
- UPnP (UPnP to MPD proxy)

3. Audio Senders
These are applications and components integrated into moOde that can send output to other devices.

- Bluetooth (send MPD output to a Bluetooth speaker)
- Multiroom (send MPD, Airplay or Spotify Connect output to multiple other moOde players to-the-ear synchronized).
