Moode Source Tree <!-- omit in toc -->
==========================================
By: Tim Curtis (c) 2022
Updated: 2022-06-12

This developer guide describes the function of directories and files in the moode Git repository.


|Directory|Description|
| :- | :- |
|.github|Setup for handling the repo at github|
|/|Github repo stuff|
|/boot|Boot config files|
|/etc|Overwrite the standard /etc directory with our defaults|
|/home|Add files for moOde ssh user|
|/lib/systemd/system|Definition of moode services|
|/sdcard/Stereo text|Sound file for stereo testing|
|/usr/lib/tmpfiles.d/mpd.conf|Default mpd configuration|
|/usr/local/bin|Command line utilities|
|/usr/local/etc|Default config files|
|/usr/local/share|Shared config files|
|/var/lib/mpd|Playlists|
|/var/local/php|Directory containing the PHP session file.|
|&nbsp;&nbsp;&nbsp;&nbsp;sess_ho7vk67sqrjua8sme0pqhsjgdq|PHP session file. The format of the filename is "sess_sessionid". The sessionid is stored in cfg_system param named 'sessionid'.|
|/var/local/www/commandw|Directory containing Scripts that support system options and the renderers.|
|&nbsp;&nbsp;&nbsp;&nbsp;lcd-updater.py|Script stub executed by the LCD updater `/var/www/util/lcd-updater.sh`.|
|&nbsp;&nbsp;&nbsp;&nbsp;restart.sh|Script that is executed by clicking the Restart or Shutdown buttons on the UI. |
|&nbsp;&nbsp;&nbsp;&nbsp;slpower.sh|Script that is executed for LMS/Squeezelite power on/off actions.|
|&nbsp;&nbsp;&nbsp;&nbsp;spotevent.sh|Script that is executed for librespot events|
|&nbsp;&nbsp;&nbsp;&nbsp;spspost.sh|Script that is executed for shairport-sync event hook `run_this_after_exiting_active_state`|
|&nbsp;&nbsp;&nbsp;&nbsp;spspre.sh|Script that is executed for shairport-sync event hook `run_this_before_entering_active_state`|
|/var/local/www/db|Directory containing sqlite3 database files|
|&nbsp;&nbsp;&nbsp;&nbsp;moode-sqlite3.db|sqlite3 database containing the configuration tables including cfg_system, cfg_radio, etc.|
|&nbsp;&nbsp;&nbsp;&nbsp;moode-sqlite3.db.sql|sqlite3 database export. This file is used in the moode-player package build to create the .db file|
|/var/local/www/imagesw|Directory containing images created via the cover/logo import options in Playlist and Radio views and the thumbnail generator `/var/www/util/thumb-gen.php` for album covers.|
|&nbsp;&nbsp;&nbsp;&nbsp;/imagesw/playlist-covers|Thumbnail covers for playlists.|
|&nbsp;&nbsp;&nbsp;&nbsp;/radio-logos|Full size logos and thumbnails for radio stations.|
|&nbsp;&nbsp;&nbsp;&nbsp;/thmcache|Thumbnail covers for albums. These JPG files are created by the Thumbnail Generator `/var/www/util/thumb-gen.php`|
|<font color="red">&nbsp;&nbsp;&nbsp;&nbsp;/toggle</font>|<font color="red">This directory contains 2 icon png files but does not appear to be used.</font>|
|/var/local/www/currentsong.txt|This file contains metadata associated with the currently playing song or station. It corresponds to the "Metadata file" option in System Config. It's updated within 3 seconds of UI metadata changes by the updExtMetaFile() function in `/var/www/daemon/worker.php`.|
|/www|Text goes here|
|/www/command|This directory contains the PHP scripts that receive and process commands sent by the front-end UI.|
|/www/css|This directory contains the Content Style Sheets for the front-end UI.|
|/www/daemon|This directory contains PHP, Python, and BASH scripts that run in the background.|
|/www/fonts|This directory contains the Open Source Lato font that serves as the application default font. <font color="red">This directory also contains a set of Flat-UI-Icon files that don't appear to be used.</font>|
|/www/images|This directory contains the Now Playing icon in both dark and white variants as well as the gray Off solid circle used in the Config template toggle buttons.|
|/www/inc|This directory contains the PHP `require_once` function libraries.|
|/www/js|Text goes here|
|/www/templates|Text goes here|
|/www/util|Text goes here|
|/www/webfonts|This directory contains the Font Awesome icons.|
