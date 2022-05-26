Master Development Guide <!-- omit in toc -->
=============================================
By: Tim Curtis (c) 2022
Updated: 2022-03-11

This guide provides information on how to get started developing for moOde.

### Table of Contents <!-- omit in toc -->

- [1. Overview](#1-overview)
- [2. The repositories](#2-the-repositories)
- [3. Reference materials](#3-reference-materials)

# 1. Overview

moOde audio player is a Web application consisting of a front-end UI written in HTML/CSS and Javascript, and a back-end wrapper for Music Player Daemon (MPD) written in PHP plus a variety of utilities written in PHP, BASH and Python. AJAX and PHP templates are used for sending data between the front-end and back-end. The data format is either $_POST associative arrays or JSON.

Bootstrap and JQuery frameworks are used for front-end implementation. The Zend Media framework is used in the back-end for extracting embedded cover art images from FLAC, MP3 and M4A audio file formats. The Web stack uses NGINX, SQLite3 and PHP-FPM.

# 2. The repositories

To start developing for moOde it's necessary to first become familiar with the source repositories on Github. They are located at (https://github.com/moode-player) and are listed below. Follow this by consulting individual reference materials listed in section 3.

|Repository|Description|
| :- | :- |
|docs|Documentation|
|moode|moOde sources and configs|
|imgbuild|Scripts to generate a custom OS image for moOde|
|pkgbuild|Scripts to build third-party packages for moOde|
|pkgsource|Sources for third-party packages used by moOde|

# 3. Reference materials

The following reference materials provide details on the structure of moOde and on how to use the Build Tooling to generate the front-end and back-end, configure and build packages and build the ISO image.

|Document|Description|
| :- | :- |
|[Source tree](https://github.com/moode-player/docs/blob/main/moode_source_tree.md)|Function of each directory and file in the moode repository source tree|
|[Development with Gulp](https://github.com/moode-player/docs/blob/main/development_with_gulp.md)|This developer guide explains how to use the `Gulp toolkit` to automate building the moOde front-end.|
|[Building Debian packages](https://github.com/moode-player/docs/blob/main/building_debian_packages.md)|This developer guide describes how `Debian` packages are used in moOde.|
|[Build the moode-player package](https://github.com/moode-player/docs/blob/main/build_moode_player_package.md)|his developer guide explains how to build the `moOde-player` package from sources.|
|[moOde audio infrastructure](https://github.com/moode-player/docs/blob/main/moode_audio_infrastructure.md)|This document describes the audio infrastructure of moOde audio player.|
