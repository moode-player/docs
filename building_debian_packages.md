Building Debian packages <!-- omit in toc -->
=================================
by @bitlab (c) 2021

This developer guide describes how `Debian` packages are used in moOde.

### Table of Contents <!-- omit in toc -->
- [1. Introduction](#1-introduction)
- [2. moOde vs Debian packages](#2-moode-vs-debian-packages)
- [3. Debian packages](#3-debian-packages)
  - [3.1. Introduction](#31-introduction)
  - [3.2. Using several package managers](#32-using-several-package-managers)
  - [3.3. Update list of available packages](#33-update-list-of-available-packages)
  - [3.4. Install package](#34-install-package)
  - [3.5. Uninstall package](#35-uninstall-package)
  - [3.6. Package information](#36-package-information)
  - [3.7. Search package](#37-search-package)
  - [3.8. List packages](#38-list-packages)
  - [3.9. Lock a package](#39-lock-a-package)
  - [3.10. Mirror and deb repo](#310-mirror-and-deb-repo)
- [4. moOde Debian repo](#4-moode-debian-repo)
  - [4.1. Introduction](#41-introduction)
  - [4.2. Install the moOde repo](#42-install-the-moode-repo)
- [5. Build packages](#5-build-packages)
  - [5.1. Introduction](#51-introduction)
  - [5.2. How to start with a new/update package for moOde](#52-how-to-start-with-a-newupdate-package-for-moode)
  - [5.3. Installing the building tools for deb files](#53-installing-the-building-tools-for-deb-files)
  - [5.4. Getting the source from a package](#54-getting-the-source-from-a-package)
  - [5.5. From source package without changes](#55-from-source-package-without-changes)
  - [5.6. Directly from source](#56-directly-from-source)
  - [5.7. Making changes](#57-making-changes)
  - [5.8. Generating from a python package](#58-generating-from-a-python-package)
  - [5.9. Generating from a rust application](#59-generating-from-a-rust-application)
  - [5.10. Binary package with FPM](#510-binary-package-with-fpm)
  - [5.11. How and where to we store the sources etc](#511-how-and-where-to-we-store-the-sources-etc)
  - [5.12. Checklist](#512-checklist)
- [6. Upload the package to the repository](#6-upload-the-package-to-the-repository)
  - [6.1. How to upload](#61-how-to-upload)
  - [6.2. Uploading binary package](#62-uploading-binary-package)
  - [6.3. Upload source package](#63-upload-source-package)


# 1. Introduction

Using packages provides the following benefits:
* Easy install/upgrade/downgrade/remove of software components.
* Use of a package manager and a package repository.
* Insight in the installed packages.
* Uniform way of providing software components.
* Automatic dependencies management (like downloading the required dependencies and preventing update/remove.
* Decouple building from the moOde release.
* Easier development of moOde releases and updates.

This guide should answer question about:
* Using the repo/packages
* Building packages
* Placing packages in the repo

For additional detailed information about Debian packaging refer to the wide variety of Internet resources that are available.

# 2. moOde vs Debian packages

moOde is base on the Debian distribution named Raspberry Pi OS (formerly Raspbian). This document is written in mind with the migration of moOde 7 series from Raspberry Pi OS (Buster) to Raspberry Pi OS (Bullseye) for moOde 8 series.

moOde exists out-of 3 major parts:
1. Raspbian OS + additional stock packages
2. Third-party software that is build for moOde, including moOde specific build options and patches.
3. moOde software

Everything in 1 is already based on packages.
The first goal is to provide packages for all third-party software of 2.
When that target is reached, maintenance of the third-party software is decoupled from moOde.
moOde then just use the packages as part of a new major/minor release. But aren't more part of the build process of a specific release.

The packages are typically stored in a so called, online, package repository.

Currently it isn't decided yet when to use packages in the repository:
1. Only during development and testing.
1. With the deployed installs. For example during an update.

Anyhow there is no explicit intention to make an `apt upgrade` safe system, to let user upgrade parts of the system it self.

A moOde is treated like an embedded system, with hand picked parts. A release comes with explicit software version used for this explicit moOde release.
To find back the package that are build for the moOde distro the suffix `~moode~bullseye` is added to Debian release number.

For example `foobar-1.2.3-1` becomes `foobar-1.2.3-1~moode~bullseye`. If new patches or build options need to change also the Debian release should be increased, like  `foobar-1.2.3-2~moode~bullseye`.



# 3. Debian packages

## 3.1. Introduction
This chapter provide an overview of the most often used commands, involved with dealing with packages.

## 3.2. Using several package managers
Debian package management is not the only package manager present on the system. Several of them are fighting to provide you the must up to date package. Most of those are specific for a certain eco system. Like pip for python and cargo for rust. And they will not collaborate together.

That is why mixing those isn't a good plan and should be avoided when possible for final moOde release (building environment is different stuff).
Also trying being to smart isn't a good plan. For example in the install script of Debian package using pip to install python dependencies.

Instead it can be needed for example to convert a pip package to Debian package.

## 3.3. Update list available packages
Before any package activities are performed, is always a good practice to update the list of available packages:
```
sudo apt update
```

## 3.4. Install package
Installing a package also results in automatically installing of the required dependencies.
```
sudo apt install yourpackagename1 yourotherpackagename
```
Or with auto confirm:
```
sudo apt install -y yourpackagename
```

Installing a package with a specific version:
```
sudo apt install yourpackagename=1.2.3
sudo apt install yourpackagename=1.2.3-1~moode~bullseye
```

Installing of a local package file:
```
sudo apt install ./yourpackagename=1.2.3-1~moode~bullseye_armhf.deb
```
Keep in mind that in this form only package dependencies can be satisfied from the repo.
If you need to install multiple local package with dependencies, you can provide multiple packages at once to the command.

## 3.5. Uninstall package
During an uninstall it is checked if no other packages depends of the to uninstall package.

```
sudo apt remove yourpackagename
```

Normally when you uninstall the configuration files are untouched.

With the '`purge' command even those will be removed:
```
sudo apt purge yourpackagename
```

## 3.6. Package information
Show information about the latest version in the repo:
```
apt info foobar
```

Show information about all versions in the repo:
```
apt info -a foobar
```

Show information about a specific version:
```
apt info foobar=1.2.3_1~moode~bullseye
```

Example output:
```
pi@bullseye:~ $ apt info caps=0.9.26-1~moode~bullseye
Package: caps
Version: 0.9.26-1~moode~bullseye
Priority: optional
Section: sound
Maintainer: Debian Multimedia Maintainers <debian-multimedia@lists.debian.org>
Installed-Size: 471 kB
Provides: ladspa-plugin
Depends: libc6 (>= 2.29), libgcc-s1 (>= 3.5), libstdc++6 (>= 5)
Breaks: libasound2-plugin-equal (<< 0.6-6)
Homepage: http://quitte.de/dsp/caps.html
Download-Size: 198 kB
APT-Manual-Installed: yes
APT-Sources: https://dl.cloudsmith.io/public/moodeaudio/m8x/deb/raspbian bullseye/main armhf Packages
Description: C* Audio Plugin Suite
 caps comprises a range of classic sound effects
 and a few more basic audio DSP units,
 signal and noise generators.
 The suite's aim is to provide impeccable sound quality,
 computational efficiency and no perceptible latency.
 .
 Documentation: http://quitte.de/dsp/caps.html
 ```

## 3.7. Search package

Search package name and description for 'keyword'
```
apt search keyword
```

Search for package starting with the name 'keyword'
```
apt search ^keyword
```

## 3.8. List packages

Show all installed packages:
```
apt list --installed
```

Show all installed packages build for moOde:
```
apt list --installed | grep ~moode~bullseye
```


## 3.9. Lock a package

A package with a lock has the following restrictions:
1. It cannot be updated
1. It cannot be installed
1. It cannot be removed

Lock a package:
```
$ apt-mark hold yourpackagename
yourpackagename set on hold.
```

Unlock a package:
```
$ apt-mark unhold yourpackagename
Canceled hold on yourpackagename.
```

Show locked packages:
```
$ apt-mark showhold
yourpackagename
```

Typical the locks could be set after completing a release and the inline update scripts could remove those temporary.

There more `apt-mark` options available, see man `apt-mark` for more info.


## 3.10. Mirror and deb repo

It can become handy to mirror the moOde deb repo.
This can be done with [apt-mirror](https://sourceforge.net/projects/apt-mirror/).
Use google for more info ;-)

# 4. moOde Debian repo

## 4.1. Introduction
Currently we use  [Cloudsmith](https://cloudsmith.io/~moodeaudio/repos/m8x) as Debian repo for moOde.

When you browse the repository you wil notice that is contains both the binary deb package and the  source dsc package (of which available).

The repository is public, readonly, available. For upload action it requires an user account and membership of the moodeaudio Cloudsmith repository.

## 4.2. Install the moOde repo
Installing the repository will make it available with commone apt tools.

```bash
curl -1sLf \
  'https://dl.cloudsmith.io/public/moodeaudio/m8x/setup.deb.sh' \
  | sudo -E bash
```
Before use make sure you have update the available package information:
```bash
sudo apt update
```

# 5. Build packages

## 5.1. Introduction
First you don't have to build from a moOde image, as long as you install a raspbian OS lite with the same base as moOde (bullseye in this case). And if building kernel stuff the same kernel version.

There are two forms of Debian packages:
* 'binary' packages
* source packages

The binary package is the type that you install. With the source packages you an (re)build the binary packages. The rebuild also becomes handy if you need to down or up port a packages.

The source package exists out of:
* The orginal unmodified source code
* A Debian subdir which contains all related files to build and pack the source code. It can also includes files like patches or configuration files.

Building from source packages is an easy process where not necessarily in depth knowledge of the package is required.

Only not all software lends itself to being packed as a source package.
For that kind of software only direct binary packages are generated. For software like Rust based applications and or python package this can be done automatically.

The source package exists out of three parts:
* source archive - contains unmodified source
* Debian archive - the Debian packing dir with build and patch instructions
* dsc -source description file, contains metadata and refers to the two above.


## 5.2. How to start with a new/update package for moOde

We first have to check which situation applies:
* A - Stock, use it as it is (stock provides binay and source package)
* B - Stock/Unstable/upstream, rebuild from source package (generates new binary and source package)
* C - Source with Debian directory, rebuild from source directory (generates new binary and source package)
* D - Source with other older/newer Debian source package available typical with an upgrade (generates new binary and source package)
* E - Only source, create a new package (generates new binary and hopefully a source package)

The action depending on the situation:
* Situation A - your done just install the stock package.
* Situation B-D - your covered with the standard tools. Home before diner .. most times.
* Situation E - we need to generate a new package. Plan depends on what the type of software.

The hardest is the first time a piece of software is packed, after that it becomes easier to maintain the package updates.

TBD
Rust Application
Python package

## 5.3. Installing the building tools for deb files

You will need at least the following tools installed:
```
sudo apt install build-essential fakeroot devscripts
```

## 5.4. Getting the source from a package

To rebuild for a source package we need the source package local. There are several ways to do it.
The most easy is as the source comes from an already installed package repository.

When the source is in a configured repository you can use the `source` command to download the three different parts and let those be unpacked to a directory with the package name:
```
apt-get source caps
```
```
apt-get source caps=0.9.26-1~moode~bullseye
```

If you just have the http/ftp location or a not configured repository and can find a link to dsc you can use the command `dget`.

In the Debian package browser you can find the dsc file on the right side of the screen For example take a look at https://packages.debian.org/bullseye/caps.

Or with the cloudsmith moodeaduio repo go to https://cloudsmith.io/~moodeaudio/repos/m8x/packages/?q=filename%3Adsc%24 and right click on the download button to copy the file link.


Both commands below wil have the same result as using `apt-get source`:
```
dget http://deb.debian.org/debian/pool/main/c/caps/caps_0.9.26-1.dsc
```
```
dget https://dl.cloudsmith.io/public/moodeaudio/m8x/deb/raspbian/pool/bullseye/main/c/ca/caps_0.9.26-1~moode~bullseye/caps_0.9.26-1~moode~bullseye.dsc
```

In the example below I will use the `dget` command.

## 5.5. From source package without changes
This not a typical case, but all other examples are more less a variant on this one.

Step 1 - get the source of the package. This requires an dsc file
```
dget http://deb.debian.org/debian/pool/main/f/foobar/foobar_1.0.0-1.dsc
cd foobar_1.0.0
```

Check if we comply to the build dependencies:
```
dpkg-checkbuilddeps
```

If missing you can install those by hand using the `apt install` command.
Or build package that installs the build deps for the current package.

This is done with:
```
mk-build-deps --install --root sudo --remove
rm foobar-build-deps_1.0.0-1_armhf.*
```

Normally is now the time to start further customization. But we will wait with that for the next example. For now we just going to build it with the following command:

```
dpkg-buildpackage -us -uc
```

If everything went well multiple files are generated in one directory up (./..).

Example list of such generated files:
```
caps_0.9.26-1~moode~bullseye_armhf.buildinfo
caps_0.9.26-1~moode~bullseye_armhf.changes
caps_0.9.26-1~moode~bullseye_armhf.deb
caps_0.9.26-1~moode~bullseye.debian.tar.xz
caps_0.9.26-1~moode~bullseye.dsc
caps_0.9.26.orig.tar.bz2
```

## 5.6. Directly from source

Sometimes software already contains a Debian subdir (like the moOde upnp subsystem).
The first time

## 5.7. Making changes

tbd :
define atonomy of the Debian folder. control, rules, copyright
adding configuration files
Debian release name
changelog
upgrading source
patches


## 5.8. Generating from a python package

tbd
## 5.9. Generating from a rust application

tbd

## 5.10. Binary package with FPM

tbd

## 5.11. How and where to we store the sources etc

<< still to be discussed ... >>
<< bitlab proposal:
I would want to avoid storing the zipped sources of archive in the moode project it self, it makes it very hard to do maintenance.

I would like to suggest to fork each third-party project as separate project in [moOde](https://github.com/moode-player). A lot of projects are in this way controlling there own dependencies.

Keep that forked/cloned third-party repos as 'clean' possible; only upstream code. Own patches or configs are part of the deb package.

Use a separate git project `mooderepo` to store the debian directories and other build stuff of thirdparty projects.

Only store the binary packages in the debian package repository.
And of course make an offline backup with something like `apt-mirror`.

>>




## 5.12. Checklist

Ok you know have a package. To be sure check if:

1. Is the correct Debian release set? Don't forget the ~moode~bullseye suffix!
2. If package was already released and in use, don't forget to increate the Debian release?
3. Are the depencies complete?
4. Is the metadata in the debian/control correct? Especially the home/credits of the orignal author
5. Is the copyright file correct?
6. When it applies are default config files supplied (not moOde specific)?

# 6. Upload the package to the repository

## 6.1. How to upload
There is a [webinterface](https://cloudsmith.io/~moodeaudio/repos/m8x) available and commandline tooling.

Install of the tooling:
```bash
sudo pip3 install --upgrade cloudsmith-cli
cloudsmith login
```

## 6.2. Uploading binary package

Upload an deb file requires using the `cloudsmith push deb` commmand, followed by the repo and the deb file to upload:
```
cloudsmith push deb moodeaudio/m8x/raspbian/bullseye ./foobar_1.0.0-1~moode~bullseye_armhf.deb
```

## 6.3. Upload source package

You have to provide all three parts of the source packge to the cloudsmith push command:
```
cloudsmith push deb moodeaudio/m8x/raspbian/bullseye ./foobar_1.0.0-1~moode~bullseye.dsc --sources-file foobar_1.0.0.orig.tar.gz  --changes-file foobar_1.0.0-1~moode~bullseye.debian.tar.xz
```
