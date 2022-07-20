Moode Source Tree <!-- omit in toc -->
==========================================
By: Tim Curtis (c) 2022
Updated: 2022-06-12

This developer guide describes the function of directories and files in the moode Git repository.


|File|Description|
| :- | :- |
|/boot|Text goes here|
|/etc|Text goes here|
|/home|Text goes here|
|/lib|Text goes here|
|/sdcard|Text goes here|
|/var|Text goes here|
|/var/lib|Text goes here|
|__/var/local/php__|Directory containing the PHP session file.|
|&nbsp;&nbsp;&nbsp;&nbsp;sess_ho7vk67sqrjua8sme0pqhsjgdq|PHP session file. The format of the filename is "sess_sessionid". The sessionid is stored in cfg_system param named 'sessionid'.|
|__/var/local/www/commandw__|Directory containing Scripts that support system options and the renderers.|
|&nbsp;&nbsp;&nbsp;&nbsp;lcd-updater.py|Script stub executed by the LCD updater `/var/www/util/lcd-updater.sh`.|
|&nbsp;&nbsp;&nbsp;&nbsp;restart.sh|Script that is executed by clicking the Restart or Shutdown buttons on the UI. |
|&nbsp;&nbsp;&nbsp;&nbsp;slpower.sh|Script that is executed for LMS/Squeezelite power on/off actions.|
|&nbsp;&nbsp;&nbsp;&nbsp;spotevent.sh|Script that is executed for librespot events|
|&nbsp;&nbsp;&nbsp;&nbsp;spspost.sh|Script that is executed for shairport-sync event hook `run_this_after_exiting_active_state`|
|&nbsp;&nbsp;&nbsp;&nbsp;spspre.sh|Script that is executed for shairport-sync event hook `run_this_before_entering_active_state`|
|__/var/local/www/db__|Directory containing sqlite3 database files|
|&nbsp;&nbsp;&nbsp;&nbsp;moode-sqlite3.db|sqlite3 database containing the configuration tables including cfg_system, cfg_radio, etc.|
|&nbsp;&nbsp;&nbsp;&nbsp;moode-sqlite3.db.sql|sqlite3 database export. This file is used in the moode-player package build to create the .db file|
|__/var/local/www/imagesw__|Directory containing images created via the cover/logo import options in Playlist and Radio views and the thumbnail generator `/var/www/util/thumb-gen.php` for album covers.|
|&nbsp;&nbsp;&nbsp;&nbsp;/imagesw/playlist-covers|Thumbnail covers for playlists.|
|&nbsp;&nbsp;&nbsp;&nbsp;/radio-logos|Full size logos and thumbnails for radio stations.|
|&nbsp;&nbsp;&nbsp;&nbsp;/thmcache|Thumbnail covers for albums. These JPG files are created by the Thumbnail Generator `/var/www/util/thumb-gen.php`|
|<font color="red">&nbsp;&nbsp;&nbsp;&nbsp;/toggle</font>|<font color="red">This directory contains 2 icon png files but does not appear to be used.</font>|
|/var/local/www/currentsong.txt|This file contains metadata associated with the currently playing song or station. It corresponds to the "Metadata file" option in System Config. It's updated within 3 seconds of UI metadata changes by the updExtMetaFile() function in `/var/www/daemon/worker.php`.|
|__/var/www/command__|This directory contains the PHP scripts that receive and process commands sent by the front-end UI.|
|__/var/www/css__|This directory contains the Content Style Sheets for the front-end UI.|
|__/var/www/daemon__|This directory contains PHP, Python, and BASH scripts that run in the background.|
|__/var/www/fonts__|This directory contains the Open Source Lato font that serves as the application default font. <font color="red">This directory also contains a set of Flat-UI-Icon files that don't appear to be used.</font>|
|__/var/www/images__|This directory contains the Now Playing icon in both dark and white variants as well as the gray Off solid circle used in the Config template toggle buttons.|
|__/var/www/inc__|This directory contains the PHP `require_once` function libraries.|
|__/var/www/js__|Text goes here|
|__/var/www/templates__|Text goes here|
|__/var/www/util__|Text goes here|
|__/var/www/webfonts__|This directory contains the Font Awesome icons.|
