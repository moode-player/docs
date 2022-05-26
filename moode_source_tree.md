Moode Source Tree <!-- omit in toc -->
==========================================
By: Tim Curtis (c) 2022
Updated: 2022-MM-DD

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
|**/var/local**||
|/php|Directory containing the PHP session file.|
|sess_ho7vk67sqrjua8sme0pqhsjgdq|PHP session file. The format of the filename is "sess_sessionid". The sessionid is stored in cfg_system param named 'sessionid'.|
|/www|Directory containing writable files that support the read-only sources in /www|
|/commandw|Scripts that support system options and the renderers.|
|lcd-updater.py|Stub script executed by the LCD updater `/var/www/util/lcd-updater.sh`.|
|restart.sh|Script that is executed by clicking the Restart or Shutdown buttons on the UI. |
|slpower.sh|Script that is executed for LMS/Squeezelite power on/off actions.|
|spotevent.sh|Script that is executed for librespot events|
|spspost.sh|Script that is executed for shairport-sync event hook: run_this_after_exiting_active_state|
|spspre.sh|Script that is executed for shairport-sync event hook: run_this_before_entering_active_state|
|/db|Directory containing sqlite3 database files|
|moode-sqlite3.db|sqlite3 database containing the configuration tables including cfg_system, cfg_radio, etc.|
|moode-sqlite3.db.sql|sqlite3 database export. This file is used in the moode-player package build to create the .db file|
|/imagesw|Directory containing images created via the cover/logo import options in Playlist and Radio views and the thumbnail generator `/var/www/util/thumb-gen.php` for album covers.|
|/playlist-covers|Thumbnail covers for playlists.|
|/radio-logos|Full size logos and thumbnails for radio stations.|
|/thmcache|Thumbnail covers for albums. These |
|<font color="red">/toggle</font>|<font color="red">This directory contains 2 icon png files but does not appear to be used.</font>|
|currentsong.txt|This file contains metadata associated with the currently playing song or station. It corresponds to the "Metadata file" option in System Config. It's updated within 3 seconds of UI metadata changes by the updExtMetaFile() function in `/var/www/daemon/worker.php`.|
|**/www**|Text goes here|
|/command|Text goes here|
|/css|Text goes here|
|/daemon|Text goes here|
|/fonts|Text goes here|
|/images|Text goes here|
|/inc|Text goes here|
|/js|Text goes here|
|/templates|Text goes here|
|/util|Text goes here|
|/webfonts|Text goes here|
