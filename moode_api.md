moOde API Reference <!-- omit in toc -->
==========================================
By: @Gjuju (c) 2026
Updated: 2026-10-26

This document lists every HTTP endpoint the moOde back-end exposes: the REST API meant for external clients, and the controllers the WebUI calls through AJAX. It replaces the REST API section that used to be in the Setup Guide.

- [1. Overview](#1-overview)
  - [1.1. Two kinds of endpoints](#11-two-kinds-of-endpoints)
  - [1.2. Request format](#12-request-format)
  - [1.3. Common behaviour](#13-common-behaviour)
- [2. REST API (command/index.php)](#2-rest-api-commandindexphp)
  - [2.1. Playback and queue](#21-playback-and-queue)
  - [2.2. Volume](#22-volume)
  - [2.3. Audio processing](#23-audio-processing)
  - [2.4. Renderers and display](#24-renderers-and-display)
  - [2.5. Multiroom](#25-multiroom)
  - [2.6. Library](#26-library)
  - [2.7. MPD commands](#27-mpd-commands)
- [3. Other CLI-safe controllers](#3-other-cli-safe-controllers)
  - [3.1. command/system.php](#31-commandsystemphp)
  - [3.2. command/dashboard.php](#32-commanddashboardphp)
- [4. WebUI controllers](#4-webui-controllers)
  - [4.1. command/playback.php](#41-commandplaybackphp)
  - [4.2. command/queue.php](#42-commandqueuephp)
  - [4.3. command/music-library.php](#43-commandmusic-libraryphp)
  - [4.4. command/playlist.php](#44-commandplaylistphp)
  - [4.5. command/radio.php](#45-commandradiophp)
  - [4.6. command/radio-browser.php](#46-commandradio-browserphp)
  - [4.7. command/renderer.php](#47-commandrendererphp)
  - [4.8. command/multiroom.php](#48-commandmultiroomphp)
  - [4.9. command/camilla.php](#49-commandcamillaphp)
  - [4.10. command/cfg-table.php](#410-commandcfg-tablephp)
  - [4.11. command/audioinfo.php](#411-commandaudioinfophp)
  - [4.12. command/peripheral.php](#412-commandperipheralphp)
- [5. Server engines](#5-server-engines)

# 1. Overview

## 1.1. Two kinds of endpoints

All endpoints live under `/var/www/command/` (and two engines under `/var/www/`). They fall into two groups, and the difference matters for anyone calling them from outside a Browser.

| Group | Controllers | PHP session | Use from curl, scripts, home automation |
|---|---|---|---|
| CLI-safe | `index.php`, `system.php`, `dashboard.php` | Opened with the session id stored in `cfg_system` | Yes |
| WebUI | every other controller | Opened from the Browser's `PHPSESSID` cookie | No |

A WebUI controller called without the Browser's cookie runs against a new, empty session. It does not fail: it reads empty settings, and any job it submits never reaches the worker. Only the CLI-safe controllers are meant for external clients, and the REST API in `index.php` is the one to build on.

There is no authentication: any host that can reach the player's web server can call any endpoint.

## 1.2. Request format

Every controller takes the command in the `cmd` query parameter.

```
http://moode/command/?cmd=COMMAND                     # REST API (index.php)
http://moode/command/CONTROLLER.php?cmd=COMMAND       # other controllers
```

The REST API takes its arguments in the same `cmd` value, separated by spaces. URL-encode it, or let curl do it:

```
curl -G -S -s --data-urlencode "cmd=set_volume -up 5" http://moode/command/
```

The other controllers take extra arguments as separate query parameters (`GET`) or form fields (`POST`), listed in each table below.

## 1.3. Common behaviour

- **Return format.** Commands that return data return JSON. Some return nothing.
- **Unknown command.** A controller answers the plain text `Unknown command`.
- **Input check.** Every controller passes its inputs through `chkVariables()` / `chkValue()`, which reject shell metacharacters, embedded shell command names and `..`. A rejected request answers HTTP 400 (redirect to `/response400.html`) and is logged in `/var/log/moode.log`.
- **Jobs.** Commands that hand work to the worker daemon answer `"job submitted"`, or `"worker busy"` if another job is still running. The result then arrives asynchronously.

# 2. REST API (command/index.php)

The REST API is the supported interface for external clients. Volume and transport commands follow whatever source is playing: MPD, an input source, or a renderer (Bluetooth, AirPlay, Qobuz Connect, Squeezelite).

## 2.1. Playback and queue

#### get_currentsong
Gets the contents of `/var/local/www/currentsong.txt`. Turn on `Metadata file` in the MPD options section of Audio Config to generate this file.  
Arguments: None  
Returns: Contents of the metadata file
```
{"file":"NAS\/TRX-FLAC\/Yes\/The Yes Album\/06 Perpetual Change.flac","artist":"Yes","album":"The Yes Album","title":"Perpetual Change","coverurl":"\/coverart.php\/NAS%2FTRX-FLAC%2FYes%2FThe%20Yes%20Album%2F06%20Perpetual%20Change.flac","track":"6","date":"197100","composer":"Anderson\/Squire","encoded":"FLAC 16\/44.1 kHz, 2ch","bitrate":"717 kbps","outrate":"PCM 16\/44.1 kHz, 2ch","volume":"15","mute":"1","state":"play"}
```

#### get_output_format
Gets the ALSA output format, or `Not playing`.  
Arguments: None  
Returns: Output format
```
{"format":"PCM 16\/44.1 kHz, 2ch"}
{"format":"Not playing"}
```

#### play_item | play_item_next
Plays the specified item.
- A Radio station or Track already in the Queue is played, otherwise it is added and played (`play_item_next` adds it after the current song).
- An Album or Playlist clears the Queue before it is added and played.

Arguments: URL, or URI relative to the MPD music or playlists root  
Returns: `{"info":"OK"}`
```
play_item RADIO/Soma FM - Fluid.pls
play_item http://mediaserv38.live-streams.nl:8006/live
play_item Default Playlist
play_item NAS/TRX-FLAC/Yes/The Yes Album
play_item NAS/TRX-FLAC/Yes/The Yes Album/03 Starship Trooper.flac
```

#### clear_queue
Clears the Queue.  
Arguments: None  
Returns: Nothing

#### toggle_play_pause
Toggles playback of the active source.
- MPD: play/stop for a Radio station, play/pause for a song file.
- Bluetooth, AirPlay, Qobuz Connect, Squeezelite: sent to the renderer.
- Other sources: handled by MPD, as above.

Arguments: None  
Returns: The resulting state. A renderer that does not report it returns `toggle` (AirPlay, Squeezelite).
```
{"state":"pause"}
{"state":"stop"}
{"state":"toggle"}
{"alert":"Playback cannot be controlled from moOde for Bluetooth"}
```

#### next | previous
Skips to the next or previous item.
- Bluetooth, AirPlay, Qobuz Connect, Squeezelite: sent to the renderer. The renderer applies its own rules (for example `previous` past the first seconds of a track restarts it).
- Other sources: relayed to MPD (see [2.7](#27-mpd-commands)).

Arguments: None  
Returns: The command sent to the renderer, or the MPD response
```
{"state":"next"}
{"0":"OK"}
```

## 2.2. Volume

The volume is set where it lives for the active source, reported as `scope`:

| Active source | MPD volume type | `scope` | Where the volume is set |
|---|---|---|---|
| MPD, Input | Hardware | `dac` | DAC mixer (`vol.sh`) |
| MPD | Software | `mpd` | MPD (`vol.sh`) |
| MPD | CamillaDSP | `camilladsp` | CamillaDSP, through MPD and `mpd2cdspvolume` |
| MPD, Input | Fixed (0dB) | `fixed` | Nowhere |
| Bluetooth, AirPlay, Qobuz Connect, Squeezelite | any | `source` | The renderer itself |
| Spotify Connect, RoonBridge, Plexamp | any | `source` | Not controllable from moOde |

During a renderer session moOde keeps the DAC and CamillaDSP at the session maximum, so only the renderer's own volume moves. Multiroom receivers follow MPD volume changes; during a renderer session they are already at their maximum.

If several renderer flags are set at once, the source is the first one in this order: Input, Bluetooth, AirPlay, Spotify, Qobuz, RoonBridge, Plexamp, Squeezelite.

#### get_volume
Gets the volume of the active source, where it lives and what can be controlled.  
Arguments: None  
Returns: Volume state

| Key | Value |
|---|---|
| `volume` | Volume 0-100, as a string. For MPD and Input it is the knob level, also when `scope` is `fixed`. `null` if a renderer does not answer. |
| `muted` | `yes` or `no` |
| `source` | `mpd`, `input`, `bluetooth`, `airplay`, `spotify`, `qobuz`, `squeezelite`, `plexamp`, `roonbridge` |
| `scope` | See the table above |
| `can_volume`, `can_mute`, `can_transport` | `true` or `false` |
| `alert` | Present when `can_volume` is `false` |

```
{"volume":"40","muted":"no","source":"mpd","scope":"dac","can_volume":true,"can_mute":true,"can_transport":true}
{"volume":"35","muted":"no","source":"qobuz","scope":"source","can_volume":true,"can_mute":true,"can_transport":true}
{"volume":"50","muted":"no","source":"airplay","scope":"source","can_volume":true,"can_mute":false,"can_transport":true}
{"volume":"40","muted":"no","source":"spotify","scope":"source","can_volume":false,"can_mute":false,"can_transport":false,"alert":"Volume cannot be changed from moOde for Spotify"}
```

#### set_volume
Sets the volume to N, up or down N, or toggles mute, on the active source.  
Arguments: `N | -up N | -dn N | -mute`, where N is digits only  
Returns: Volume state, as `get_volume`, or an alert
```
{"volume":"45","muted":"no","source":"mpd","scope":"dac","can_volume":true,"can_mute":true,"can_transport":true}
{"alert":"Invalid volume command"}
{"alert":"Volume cannot be changed when MPD volume type is Fixed (0dB)"}
{"alert":"Volume cannot be muted from moOde for AirPlay"}
```

## 2.3. Audio processing

#### get_cdsp_config
Gets the current CamillaDSP config name.  
Arguments: None  
Returns: Config name or `off`
```
{"config":"off"}
```

#### set_cdsp_config
Sets CamillaDSP to the specified config.  
Arguments: A config name from the list of available configs, including `Off`  
Returns: Config name, or an alert
```
{"config":"V2-ProtoDAC.yml"}
{"alert":"Invalid Configuration name"}
```

#### get_eq_status
Gets the Graphic EQ or Parametric EQ status.  
Arguments: None  
Returns: `eq_type|curve_index|curve_name` or `Off`
```
{"status":"none|0|Off"}
{"status":"graphic|3|Lo Boost Plus"}
```

#### set_eq_curve
Sets the Graphic EQ or Parametric EQ to a curve name, a curve index or `Off`.  
Arguments: `graphic | parametric` followed by `curve_name | curve_index | Off`  
Returns: Status, or an error
```
{"status":"graphic|3|Lo Boost Plus"}
{"error":"Disable CamillaDSP first"}
```

## 2.4. Renderers and display

#### restart_renderer
Restarts the specified renderer.  
Arguments: `--bluetooth | --airplay | --spotify | --qobuz | --upnp | --squeezelite | --plexamp | --roonbridge` (only the renderers enabled on this build)  
Returns: Confirmation, or an alert
```
{"info":"Renderer restart submitted"}
{"alert":"Missing or invalid argument"}
```

#### renderer_onoff
Turns the specified renderer on or off.  
Arguments: a renderer, as above, followed by `on | off`  
Returns: Confirmation, or an alert
```
{"info":"Renderer --airplay on submitted"}
```

#### set_coverview
Turns the CoverView screen saver on or off.  
Arguments: `-on | -off`  
Returns: Resulting state
```
{"info":"CoverView off"}
```

#### set_display
Sets the locally attached display to the WebUI or Peppy, or toggles between them.  
Arguments: `webui | peppy | toggle`  
Returns: Confirmation
```
{"info":"Set display to peppy submitted"}
```

## 2.5. Multiroom

#### get_receiver_status
Gets the status of this player's multiroom receiver.  
Arguments: None  
Returns: `rx,On|Off|Disabled|Unknown,volume,mute_1|0,mastervol_opt_in_1|0,hostname,multicast_addr`
```
{"status":"rx,Off,15,0,1,sig,239.0.0.1"}
```

#### set_receiver_onoff
Turns this player's multiroom receiver on or off.  
Arguments: `-on | -off`  
Returns: Receiver status, or an alert if it is already in that state
```
{"status":"rx,On,15,0,1,sig,239.0.0.1"}
{"alert":"Receiver is already On"}
```

#### trx_control
Internal: used by the renderer event scripts and by a multiroom sender to drive its receivers. Passes up to three arguments to `/var/www/util/trx-control.php`.  
Returns: The raw result, not JSON

## 2.6. Library

#### upd_library
Submits a library update.  
Arguments: None  
Returns: Confirmation
```
{"info":"Library update submitted"}
```

## 2.7. MPD commands

Any other command is sent as is to MPD, and the response is returned as JSON. See the [MPD protocol](https://mpd.readthedocs.io/en/latest/protocol.html) for the commands and their results. `play`, `pause` and `stop` always go to MPD, also during a renderer session.
```
curl -G -S -s --data-urlencode "cmd=status" http://moode/command/
```

# 3. Other CLI-safe controllers

These two controllers open the stored session like the REST API, so they work from curl.

## 3.1. command/system.php

| cmd | Arguments | Returns |
|---|---|---|
| `reboot` | — | Job: `"job submitted"` or `"worker busy"` |
| `poweroff` | — | Job |
| `update_library` | — | Job |
| `get_client_ip` | — | The caller's IP address |
| `restart_local_display` | — | Job, only if a local display is configured |

## 3.2. command/dashboard.php

Used by the multi-player Dashboard to drive another moOde player.

| cmd | Arguments | Returns |
|---|---|---|
| `update_library`, `regen_thmcache` | — | Job |
| `regen_thumbnails`, `refresh_screen` | — | Nothing; refreshes the WebUI |
| `play`, `pause`, `stop`, `next`, `prev` | — | Nothing; runs `mpc` directly, MPD only |

# 4. WebUI controllers

These controllers serve the WebUI and rely on the Browser's session cookie (see [1.1](#11-two-kinds-of-endpoints)). They are listed for contributors; external clients should use the REST API.

`GET` arguments go in the query string, `POST` arguments in the request body.

## 4.1. command/playback.php

| cmd | Arguments | Returns |
|---|---|---|
| `upd_volume` | POST `volknob`, `event` (`mute`, `unmute` or a change) | `"OK"`; sets the knob volume and multiroom receivers |
| `mute_rx_vol` | — | `"OK"`; toggles mute on the multiroom receivers |
| `toggle_ashuffle` | POST `toggle_value` (`0`/`1`) | Nothing; stops or starts auto-shuffle |
| `toggle_coverview` | — | Nothing; toggles CoverView |
| `upd_toggle_coverview` | POST `toggle_value` | Nothing |
| `get_mpd_status` | — | MPD status |
| `get_play_history` | — | Lines of the playback history log |
| `reset_screen_saver`, `upd_clock_radio` | — | Job |
| `set_bg_image` | POST `blob` | Job |
| `remove_bg_image` | — | `"OK"` |

## 4.2. command/queue.php

When auto-shuffle is on, the commands that change the Queue turn it off first.

| cmd | Arguments | Returns |
|---|---|---|
| `get_playqueue` | — | Queue items |
| `get_playqueue_item` | GET `songpos` | File path of that item |
| `get_playqueue_item_tag` | GET `songpos`, `tag` | Value of that tag |
| `delete_playqueue_item` | GET `range` | Nothing |
| `move_playqueue_item` | GET `range`, `newpos` | Nothing |
| `clear_playqueue` | — | Nothing |
| `crop_playqueue` | — | Nothing; keeps only the current item |
| `add_item`, `add_item_next` | POST `path` | Nothing |
| `play_item`, `play_item_next` | POST `path` | Nothing; plays it if already queued |
| `clear_play_item` | POST `path` | Nothing; clears the Queue, adds and plays |
| `add_group`, `add_group_next` | POST `path[]` | Nothing |
| `play_group`, `play_group_next` | POST `path[]` | Nothing |
| `clear_play_group` | POST `path[]` | Nothing |

## 4.3. command/music-library.php

| cmd | Arguments | Returns |
|---|---|---|
| `load_library` | — | The library cache (JSON) |
| `update_library` | GET `path` (optional) | Job |
| `lsinfo` | GET `path` (optional) | Folder contents |
| `search` | GET `tagname` (`any` or `specific`), `query` | Matching items |
| `get_dbupdate_count`, `get_dbregen_count`, `get_dbanalyze_count` | — | Counter |
| `clear_libcache_all`, `clear_libcache_filtered` | — | Nothing |
| `get_saved_searches` | — | Saved search names |
| `activate_saved_search` | POST `name` | The saved search |
| `create_saved_search` | POST `name` | Job |
| `delete_saved_search` | POST `name` | Nothing |
| `clear_active_search` | — | Nothing |
| `get_recent_playlist` | — | Name of the recently added playlist |
| `thumcache_status` | — | Thumbnail cache status text |

## 4.4. command/playlist.php

| cmd | Arguments | Returns |
|---|---|---|
| `get_playlists` | — | Playlists |
| `get_playlist_contents` | POST `path` | Playlist items |
| `get_pl_items_fv` | GET `path` | Playlist items, for Folder view |
| `new_playlist`, `upd_playlist` | POST `path[name]`, `path[genre]`, `path[items][]` | Nothing |
| `add_to_playlist` | POST `path[playlist]`, `path[items][]` | Nothing |
| `del_playlist` | POST `path` | Nothing |
| `save_queue_to_playlist` | GET `name` | MPD response |
| `export_playlist` | GET `name` | The `.m3u` file, as a download |
| `analyze_import` | POST `content`, `remap` (JSON) | Analysis: `status`, `total`, item counts |
| `import_playlist` | POST `content`, `name`, `remap`, `drop_invalid` | `status` and the playlist name, or an error |
| `set_plcover_image` | POST `name`, `blob` | Job |
| `get_favorites_name`, `set_favorites_name` | GET `name` (set) | Favorites playlist name |
| `add_item_to_favorites` | GET `item` | Nothing |

## 4.5. command/radio.php

| cmd | Arguments | Returns |
|---|---|---|
| `get_stations` | — | Stations |
| `get_station_contents` | POST `path` (`RADIO/<name>.pls`) | Station fields |
| `new_station`, `upd_station` | POST `path[name]`, `path[url]`, `path[id]` and the other station fields | Validation message |
| `del_station` | POST `path` | Nothing |
| `set_ralogo_image` | POST `name`, `blob` | Job |
| `put_radioview_show_hide` | POST `block`, `type` | Nothing |
| `mpd_monitor_svc` | POST `svc` (`On`/`Off`), `opt` | Nothing; starts or stops the MPD stream monitor |
| `get_radiocover_url` | GET `title`, `station` | Cover URL for the current title |
| `clear_radiocover_url_cache` | — | Nothing |

## 4.6. command/radio-browser.php

Every command returns `{"success":true|false, ...}`. Arguments come from the query string or the form (`url`, `name`, …); the station commands also accept a JSON body with the station fields (`url`, `name`, `favicon`, `country`, `language`, `state`, `tags`, `bitrate`, `codec`, `homepage`, `stationuuid`).

| cmd | Arguments | Returns |
|---|---|---|
| `search` | `name`, `countrycode`, `tag`, `offset`, `limit` | `stations` |
| `countries`, `genres` | — | `countries` or `genres` |
| `recently_played` | — | `stations` |
| `play` | station | `message`, `name`, `url`, `format`, `bitrate`, `home_page` |
| `add`, `remove` | station | `message`; adds it to or removes it from Favorites |
| `register`, `check_registered` | station or `url` | `message` |
| `remove_recent` | station | `message` |
| `logo` | `url` | The station logo image |
| `clear_recents`, `clear_caches`, `check_servers` | — | Script result |

## 4.7. command/renderer.php

| cmd | Arguments | Returns |
|---|---|---|
| `disconnect_renderer` | POST `job` (renderer service, e.g. `slsvc`, `pasvc`, `multiroom_rx`) | Job |
| `get_aplmeta`, `get_spotmeta`, `get_qbzmeta`, `get_btmeta` | — | The renderer's metadata cache (AirPlay, Spotify, Qobuz, Bluetooth) |
| `bt_pair_response` | POST `id`, `accepted`, `code` | `"ok"` or `"failed"`; answers a Bluetooth pairing request |

## 4.8. command/multiroom.php

Sender side: drives the receivers found by discovery.

| cmd | Arguments | Returns |
|---|---|---|
| `get_rx_status` | — | Receiver statuses, separated by `:` (format as [get_receiver_status](#get_receiver_status)) |
| `set_rx_status` | POST `item` (receiver index) and one of `onoff`, `volume`, `mute` | `"OK"` |
| `tx_adv_toggle`, `rx_adv_toggle` | POST `adv_toggle` | Nothing; WebUI state |

## 4.9. command/camilla.php

| cmd | Arguments | Returns |
|---|---|---|
| `cdsp_set_config` | POST `cdspconfig` | Nothing; the WebUI is notified when done |
| `cdsp_get_config_desc` | GET `selected_config` | Config description |

## 4.10. command/cfg-table.php

| cmd | Arguments | Returns |
|---|---|---|
| `get_cfg_system` | — | `cfg_system` |
| `get_cfg_system_value` | GET `param` | One `cfg_system` value |
| `upd_cfg_system` | POST `<param>=<value>`, one or more | `"OK"` |
| `get_theme_name` | GET `theme_name` (optional) | One theme, or all |

## 4.11. command/audioinfo.php

| cmd | Arguments | Returns |
|---|---|---|
| `station_info` | GET `path` (station URL) | Station fields, as a list of `{label: value}` |
| `track_info` | GET `path` (file) | Track tags, as a list of `{label: value}` |

## 4.12. command/peripheral.php

| cmd | Arguments | Returns |
|---|---|---|
| `get_peppy_meter_list`, `get_peppy_spectrum_list` | GET `selected_folder` | Peppy Meter or Spectrum skins in that folder |

# 5. Server engines

The WebUI keeps two long-poll connections open. They are internal and not meant for external clients.

| Endpoint | Purpose |
|---|---|
| `engine-mpd.php?state=<state>` | Returns the MPD status and enhanced metadata. If `state` matches the current state, waits on MPD `idle` until something changes. Errors: `1001` Socket open failed, `1002` Socket timed out. |
| `engine-cmd.php` | Waits for a command the back-end pushes to the front-end (`sendFECmd()`), for example `inpactive1` or `refresh_screen`, and returns it. |
