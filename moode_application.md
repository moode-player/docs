Moode Application <!-- omit in toc -->
==========================================
By: Tim Curtis (c) 2023
Updated: 2023-07-11

This document describes the architecture of the moOde audio player application and how component parts are related to one another.

- [1. Overview](#1-overview)
- [2. Web page structure](#2-web-page-structure)
- [3. FE/BE communications](#3-fe-be-communications)

# 1. Overview

# 2. Web page structure

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

Lastly a few maintenance items to take note of:

1. Several JS/CSS libraries including Bootstrap, Font Awesome and jQuery should be bumped at some point but the effort to test and debug these lib upgrades is substantial and thus its a low priority.

2. A substantial part of the codebase was refactored a while back. Whats remaining is the JS but due to it's complexity and similar to the libs above the effort to test and debug is substantial and thus its a low priority.

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
