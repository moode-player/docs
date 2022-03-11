Build the moode-player package <!-- omit in toc -->
===================================================
By: @bitlab (c) 2021
Updated: 2022-03-10

This developer guide explains how to build the `moOde-player` package from sources.

### Table of Contents <!-- omit in toc -->

- [1. Pre conditions](#1-pre-conditions)
- [2. Building the package](#2-building-the-package)
  - [2.1. Where and how to make changes to the package](#21-where-and-how-to-make-changes-to-the-package)
    - [2.1.1. build.sh](#211-buildsh)
    - [2.1.2. postinstall.sh](#212-postinstallsh)
    - [2.1.3. Background information:](#213-background-information)
  - [2.2. Build options](#22-build-options)
  - [2.3. Building](#23-building)
- [3. Installing the moode-player package](#3-installing-the-moode-player-package)
  - [3.1. Local Installation on a fresh running debian image](#31-local-installation-on-a-fresh-running-debian-image)
  - [3.2. Installation from a repo on a fresh running debian image :](#32-installation-from-a-repo-on-a-fresh-running-debian-image-)

# 1. Pre conditions
Assumed you performing the build from a running Raspberry Pi OS bullseye.

You should have both the moode main source and the pkgbuild repo cloned.
```bash
git clone https://github.com/moode-player/moode.git
git clone https://github.com/moode-player/pkgbuild.git
```
***For bullseye some changes are required to the moode project itself, because php is bumped from 7.3 to version 7.4 and some services had another name.***


You have the following enviroment settings set:
```bash
export DEBFULLNAME=Foo
export DEBEMAIL=foo@bar.org
export MOODE_DIR=~/yourpathto/moode
```

# 2. Building the package

## 2.1. Where and how to make changes to the package

The current content and actions of the package can be found it the follow two scripts:

* `pkgbuild/packages/moode-player/build.sh`
* `pkgbuild/packages/moode-player/postinstall.sh`



### 2.1.1. build.sh

This one creates the package and does the following:

1. Responsible for what files are copied by coping the required files to a fake root dir (`$PKG_ROOT_DIR`).
1. List dependencies (multiple `fpm --depends` argument) to be installed.
1. Create a package (with fpm ) containing the files from the fake root dir (the fpm command on the bottom of the file)

Almost all configuration files aren't copied to /etc and /lib but to /usr/share/moode-player.

### 2.1.2. postinstall.sh

This one is included by the package and executed after install as part of the deb install process (`fpm --after-install` argument).

This file should fix things like:
* file rights
* disable/enable services
* create links
* create dir structure / touch files etc
* update setting in files etc
* when needed copy files `/usr/share/moode-player` to the correct location in `/etc|/lib`. This part is far from finsished.

### 2.1.3. Background information:
Especially the last part is far from finished. I followed for the other stuff most of mosbuild script, but not everything in the mosbuild is needed any more. (Attached it the mosbuild status, everything that is made comment is ... covered I think ;-)

A lot of the third-party packages provide now their own defaults files and or file structure  (and by this not always correct for moode). But how more we can adapt to that the less painful software upgrades and migration to new distro version will become.

Package don't allow that a files is owned by two packages. That is why I copies the config files to /usr/share/moode-player. Then we can updates postinstall.sh to copy those, selective, to the correct location. Also updates to files in /etc are treated different then in other locations.

But when a config (for example because the default is already ok) is not needed we shouldn't copy it.
And when possible I prefer changing (with sed or so) the default config file as opposed to replacing it. But that is only possible when a limited number of changes are required.

In case we need to 'own' an already existing file we can use the `dpkg-divert` command.



## 2.2. Build options
Default (***at least it should become default, currently it is turned off see the two vars below***) it also:
1. if not already present installs npm for building the front end app
1. installs clean required npm modules (in $MOODE_DIR/.npm_modules)
1. build the frontend app

Step 2 and 3 can take some time.

You can change this in `build.sh` by changing the two vars below to 0.
```bash
# sync required npm modules for gulp build
NPM_CI=0
# build web app with gulp
BUILD_APP=0
```

## 2.3. Building
The build process install required dependencies the first it runs.

```
cd pkgbuild/packages/moode-player
./build.sh
```

# 3. Installing the moode-player package
## 3.1. Local Installation on a fresh running debian image

The current package is prepared to be updated or reinstalled, this will come when we have a working package.
To be sure it you ar developing first remove the package:

```bash
sudo apt remove moode-player
```

Installation
```bash
sudo apt update
sudo apt upgrade

sudo apt install ./dist/binary/moode-player_*_armhf.deb
sudo reboot
```

When install

## 3.2. Installation from a repo on a fresh running debian image :

If the package is deployed to Cloudsmith you can installed as below:

```bash
curl -1sLf \
'https://dl.cloudsmith.io/public/moodeaudio/m8x/setup.deb.sh' \
| sudo -E bash

sudo apt update
sudo apt upgrade

sudo apt install moode-player
sudo reboot
```
