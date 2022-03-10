Development with Gulp <!-- omit in toc -->
==========================================
By: @bitlab (c) 2021
Updated: 2022-03-10

This developer guide explains how to use the `Gulp toolkit` to automate building the moOde front-end.

### Table of Contents <!-- omit in toc -->

- [1. Introduction](#1-introduction)
- [2. TL;DR](#2-tldr)
  - [2.1. Back and front end stuff](#21-back-and-front-end-stuff)
  - [2.2. You do front-end stuff](#22-you-do-front-end-stuff)
- [3. Gulp](#3-gulp)
  - [3.1. Features of Gulp with moOde](#31-features-of-gulp-with-moode)
  - [3.2. Relation to non-Gulp workflow](#32-relation-to-nongulp-workflow)
  - [3.3. Configuration](#33-configuration)
  - [3.4. About Gulp tasks](#34-about-gulp-tasks)
  - [3.5. The build and deploy process](#35-the-build-and-deploy-process)
  - [3.6. Complete available task list](#36-complete-available-task-list)
  - [3.7. Temporary files](#37-temporary-files)
- [4. Prepare development host](#4-prepare-development-host)
  - [4.1. Install required tooling](#41-install-required-tooling)
  - [4.2. Checkout out work git repo](#42-checkout-out-work-git-repo)
  - [4.3. Setup Gulp dependencies](#43-setup-gulp-dependencies)
- [5. Scenarios](#5-scenarios)
  - [5.1. Prepare a release build on a Raspberry Pi](#51-prepare-a-release-build-on-a-raspberry-pi)
  - [5.2. Front-end development on a workstation](#52-frontend-development-on-a-workstation)
  - [5.3. Front-end and back-end development on a Raspberry Pi](#53-frontend-and-backend-development-on-a-raspberry-pi)
- [6. Known issues](#6-known-issues)
  - [6.1. Gulp not found](#61-gulp-not-found)
  - [6.2. Slow Gulp web server](#62-slow-gulp-web-server)
  - [6.3. After deploy still old version in deploy location](#63-after-deploy-still-old-version-in-deploy-location)
  - [6.4. Error  ‘System limit for number of file watchers reached’ on watch](#64-error-system-limit-for-number-of-file-watchers-reached-on-watch)
- [7. Appendix](#7-appendix)
  - [7.1. Script to setup development environment](#71-script-to-setup-development-environment)

# 1. Introduction
[Gulp](https://gulpjs.com/) is used to automate and make development easier with the following use cases in mind:

1. Front-end and back-end development on the Raspberry Pi.
1. Prepare the build of a release on the Raspberry Pi.
1. Front-end development on a workstation such as Windows, Mac or Ubuntu

# 2. TL;DR
## 2.1. Back and front-end stuff

Assumptions:

- You have a Raspberry Pi running moOde 7 or higher
- You are logged in with an SSH client to the Pi

You do:

~ $ cd ~  
~ $ sudo apt-get update  
~ $ sudo apt-get -y install npm  
~ $ git clone https://github.com/moode-player/moode.git  
~ $ cd moode  
~/moode $ npm ci  
~/moode $ node_modules/.bin/gulp build  
~/moode $ node_modules/.bin/gulp deploy --test  
~/moode $ sudo mv /var/www /var/www.org  
~/moode $ sudo node_modules/.bin/gulp deploy  
~/moode $ moodeutl -r

Result:

- moOde front-end running with bundled and minified sources from /var/www/

## 2.2. You do front-end stuff

Assumes:

- You have a Raspberry Pi running moOde 7 or higher
- You can do front-end work on either your workstation or the Raspberry Pi
- You have read issue 6.2 `Slow Gulp web server` and made sure that  moode.test points to the correct ip.

You do:

~ $ cd ~  
~ $ sudo apt-get update  
~ $ sudo apt-get -y install npm  
~ $ git clone https://github.com/moode-player/moode.git  
~ $ cd moode  
~/moode $ npm ci  
~/moode $ node\_modules/.bin/gulp watch --build

Result:

- You can test the moOde build with bundles on your workstation at http://localhost:3000 or Pi at http://moode:3000.
- The moment you make a front-end source change, the files will be rebuilt.

# 3. Gulp

## 3.1. Features of Gulp with moOde
The Gulp project file ~/moode/gulpfile.js which is similar to a Make file contains the logic and targets that control what happens during a Gulp build.

- Minifying javascript, html and css.
- Combining multiple files to a bundle file. Based on markup found in header.php.
- Providing javascript source maps.
- Patch the header.php to use those bundles.
- Web Server with proxy to redirect backend calls. Automatic reload when source changes.
- Create a ‘stub’ index.html for frontend development with the builtin webserver.
- Deploy a build (front+backend) to a destination directory like /*var/*www.
- Way to update the headers of deploy files with correct date and version.

Effect:

- No longer required to generate minified files by hand. Always in sync, probably no need to check in .min files any more.
- Minified files are still ‘readable’ in the browser debugger, because the source maps are provided.
- Create automatic efficient bundles. This will reduce the initial loading time of resources.
- Automatic copy (deploy) of your development files to the /var/www directory.
- Quick roll out changes to your local test target.
- Easy workflow for front-end developers, even on their own workstation.

## 3.2. Relation to non-Gulp workflow

If the Gulp workflow isn’t used, it will not sit in the way of the non-Gulp workflow(\*). You can still use the Gulp workflow for development (and check-in the source changes) and do the final deployment in the old way.

(\*) That means no automatic minifying and no use of bundling. Plain copy of the ~/moode/www is used as deployment to /var/www/.

## 3.3. Configuration
The ~moode/package.json contains the configuration of the Gulp build options.

Normally you don’t need to touch this file, only the version number will change.

The defaults for the used paths are.

|Setting|Description|Default values|
| :- | :- | :- |
|app.src |Source location|www|
|app.cache|Cache location|build/cache|
|app.dest|Build output|build/develop|
|app.dist|Temporary location for test deploy|build/distr/var/www|
|app.deploy|Actual deploy location|*/var/www*|

## 3.4. About Gulp tasks

Gulp performs it's work by executing tasks. Tasks are specified by name and with optional arguyments.

For example:

``~/moode $ node\_modules/.bin/gulp <task\_name> [optional\_arguments]``

When executed the output will list which tasks are executed.

Main build targets:

|Task|Description|
| :- | :- |
|gulp clean [--all] |Empties the build/build and build/distr directory, with the --all also the cache is flushed.|
|gulp build [--force] |Build frontend from {app.src} and put output in {app.dest}|
|gulp watch [--build] [--force]|<p>Start web server with as root {app.src} with proxy to moode, used for local web development.</p><p>` `When --build is given, also perform a build and use {app.dest} as web root.</p>|
|gulp deploy [--test] [--all]|<p>` `Deploys everything needed (inc php etc) {app.deploy}.</p><p>With the option --test deploy to build/dist (app.dist).</p><p>` `When used to real don't forget to sudo first and do a moodeutl -r afterwards.</p><p>This not only includes the javascript, html and css stuff, but everything in the /var/www. Including patched header and footer.</p><p>With the --all option also moodeutl and stuff in the var/local/www is deployed. You db is left intact.</p><p>With deploy files are created or updated, not deleted.</p>|

Default: most tasks only update/use files that are changed (= is newer).  With `force` the files are always updated.

## 3.5. The build and deploy process

The following moOde source files are change during the deploy:

- header.php
- footer.php
- indextpl.html

Note that the file `header.php` plays an important role in specifying which resources should be processed. The resources are identified by special comments with markup instructions.

For example:
```
<!-- build:css css/styles.min.css -->
`    `<link href="css/bootstrap.min.css" rel="stylesheet">
`    `<link href="css/bootstrap-select.min.css" rel="stylesheet">
`    `<link href="css/flat-ui.min.css" rel="stylesheet">
`    `<link href="css/jquery.pnotify.default.min.css" rel="stylesheet">
`    `<link href="css/fontawesome-moode.min.css" rel="stylesheet">
`    `<link href="css/panels.min.css" rel="stylesheet">
`    `<link href="css/moode.min.css" rel="stylesheet">
<!-- endbuild -->
```
This specifies that the contained resources are minified with source maps and are replaced by the bundle file styles.min.css. Minification reduces the resources needed by the Browser to render html/css.

Other markup instructions comment, remove or uncomment code blocks.

There are also index.html files for use in combination with the Gulp web server:

- ~moode/www/index.html – includes individual resources
- ~moode/build/develop/index.html – includes resource bundles

## 3.6. Complete available task list

You need only a small number of tasks, but for the overview lets see which tasks are present:

`~/moode $ node_modules/.bin/gulp --tasks`
```
[10:57:03] Tasks for Tasks for ~/moode/gulpfile.js
[10:57:03] ├── listplugins
[10:57:03] ├── sass
[10:57:03] ├── browserSync
[10:57:03] ├── browserReload
[10:57:03] ├── cache
[10:57:03] ├── maps
[10:57:03] ├─┬ bundle
[10:57:03] │ └─┬ <series>
[10:57:03] │   ├── cache
[10:57:03] │   ├── maps
[10:57:03] │   └── <anonymous>
[10:57:03] ├── genindexdev
[10:57:03] ├── genindex
[10:57:03] ├── patchheader
[10:57:03] ├── patchfooter
[10:57:03] ├── patchindex
[10:57:03] ├── patchconfigs
[10:57:03] ├── minifyhtml
[10:57:03] ├── artwork
[10:57:03] ├── clean
[10:57:03] ├─┬ build
[10:57:03] │ └─┬ <series>
[10:57:03] │   ├── sass
[10:57:03] │   ├── genindexdev
[10:57:03] │   ├── genindex
[10:57:03] │   ├─┬ bundle
[10:57:03] │   │ └─┬ <series>
[10:57:03] │   │   ├── cache
[10:57:03] │   │   ├── maps
[10:57:03] │   │   └── <anonymous>
[10:57:03] │   ├── artwork
[10:57:03] │   └── <anonymous>
[10:57:03] ├── deploymoodeutl
[10:57:03] ├── deployvarlocalwww
[10:57:03] ├─┬ deployback
[10:57:03] │ └─┬ <series>
[10:57:03] │   ├── patchheader
[10:57:03] │   ├── patchfooter
[10:57:03] │   ├── patchindex
[10:57:03] │   ├── patchconfigs
[10:57:03] │   ├── minifyhtml
[10:57:03] │   └── <anonymous>
[10:57:03] ├── deployfront
[10:57:03] ├── upload2remote
[10:57:03] ├── deploy2remote
[10:57:03] ├─┬ deploy
[10:57:03] │ └─┬ <series>
[10:57:03] │   ├── deployfront
[10:57:03] │   ├─┬ deployback
[10:57:03] │   │ └─┬ <series>
[10:57:03] │   │   ├── patchheader
[10:57:03] │   │   ├── patchfooter
[10:57:03] │   │   ├── patchindex
[10:57:03] │   │   ├── patchconfigs
[10:57:03] │   │   ├── minifyhtml
[10:57:03] │   │   └── <anonymous>
[10:57:03] │   └── <anonymous>
[10:57:03] └─┬ watch
[10:57:03]   └─┬ <series>
[10:57:03]     ├── sass
[10:57:03]     ├── genindexdev
[10:57:03]     ├── browserSync
[10:57:03]     └── <anonymous>
```
## 3.7. Temporary files

During running tasks a directory called ~/moode/build is created. It contains 3 important subdirectories as listed below.

- cache - A cache of the minified files including the map files.
- develop – Output of a build. Contains bundles and also files to run the proxy web server.
- dist – Output of a test deploy

Overview of the contents of the build directory.

`~/moode $ tree -d build`
```
build

├── cache
│   ├── css
│   ├── js
│   └── maps
│       ├── css
│       └── js
├── develop
│   ├── css
│   ├── fonts
│   ├── images
│   │   └── toggle
│   ├── js
│   ├── maps
│   │   ├── css
│   │   └── js
│   └── webfonts
└── dist
`    `└── var
`        `└── www
`            `├── command
`            `├── css
`            `├── fonts
`            `├── images
`            `├── inc
`            `├── js
`            `├── maps
`            `│   ├── css
`            `│   └── js
`            `├── templates
`            `└── webfonts
```
# 4. Prepare development host

## 4.1. Install required tooling

Before you can use Gulp it must be installed and configured. Like most modern Web tooling Gulp is built using Javascript and requires installation of npm and node.

For installation on Raspberry Pi and other flavors of Linux that support apt-get run the command below. For other platforms see <https://nodejs.org/en/download/>.

`~ $ sudo apt-get install npm`

## 4.2 Checkout out work git repo

Probably you will use your own repo here, but for demonstration of the Gulp tooling this ok.

`~ $ git clone <https://github.com/moode-player/moode.git>`

For the rest of the document we assume that the checkout path is `~/moode`

## 4.3. Setup Gulp dependencies

The tools required for using Gulp are installed as self contained tools in `~/moode/node_modules`. Instead of installing modules with the help of the package.json, package-lock.json is used. This ensures that also the deps of deps etc., are frozen to a certain version.

With the folling command those modules are installed.

`~/moode $ npm ci`

This will take some time. Note that the node_modules directory is excluded in the `.gitignore` file so it will not be processed as part of development. The node_modules directory doesn’t need to be reinstalled if you switch branches.

Because Gulp itself is also installed as a package, running Gulp requires a path to its binary. Let’s check the installation is successful:
```
~/moode $ node_modules/.bin/gulp -v
CLI version: 2.2.1  
Local version: 4.0.2
```
To avoid specifying the path you can install Gulp global or add the node_modules/.bin directory to your search path.

# 5. Scenarios

In this section we take a look at several different ways to use Gulp.

The basic idea is that you always work with the checked out source and build/deploy the changed files.

I prefer using an IDE like VS Code (runs on Mac, Windows and Linux) which make it possible to also directly edit files, commit and run commands on a remote target as Raspberry Pi.

## 5.1 Prepare a release build on a Raspberry Pi

Here we bring a local branch in sync and do a deploy to the /var/www/ directory to validate the final release. We do a force to make sure newer files in the destination are overwritten even if newer.
```
~moode $ git fetch; git pull
~moode $ node\_modules/.bin/gulp build
~moode $ sudo node\_modules/.bin/gulp deploy --force
~moode $ moodeutl -r
```
Optionally we could have done a test deploy before the real deploy. Inspect the output in the build/dist/ directory

`~moode $ sudo node\_modules/.bin/gulp deploy –test`

## 5.2. Front-end development on a workstation

Here we assume the workstation is running Windows, Mac or Ubuntu. The
target Raspberry Pi should be running and contain a valid matching backend code.

The workflow is like:

- Start with Gulp using a local development web server with proxy backend calls to the target Raspberry Pi
- Edit source in your favorite IDE
- Save the file
- Gulp sees the changes and triggers a rebuild
- The browser is automatically reloaded (and css changes are visible)

The steps with gulp:
```
~moode $ git fetch; git pull
~moode $ node\_modules/.bin/gulp watch –build
```
With the –build you will use the build with bundle files else without bundles.

Add the end of the Gulp output you will see something like:
```
[Browsersync] Access URLs:
----------------------------------
      Local: http://localhost:3000
     External: http://0.0.0.0:3000
----------------------------------
         UI: http://localhost:3001
UI External: http://localhost:3001
----------------------------------
[Browsersync] Serving files from: build/develop
```
Which indicates that the development Web server can be reached at [http://localhost:3000](http://localhost:3000/).

Bt default it will try to open a browser for you.

An important question is unanswered: how does the development web server proxy know how the raspberry pi is reachable?

It assumes that your raspberry is reachable as [http://moode.test](http://moode.test/). (see package.json). You can change that setting in the package.json.

## 5.4. Front-end and back-end development on the Raspberry Pi

In this case both front-end (HTML/CSS/JS) and back-end (PHP/BASH/Python) development is being done on a Raspberry Pi (target).

Every time you make a change to a back-end file that needs to deployed for test the following is done:

First time:
```
~moode $ node\_modules/.bin/gulp build
~moode $ sudo node\_modules/.bin/gulp deploy --force
~moode $ moodeutl -r
```
Next time:
```
~moode $ sudo node\_modules/.bin/gulp deploy
~moode $ moodeutl -r
```
For frontend changes you could also do something like:
```
~moode $ node\_modules/.bin/gulp build
~moode $ sudo node\_modules/.bin/gulp deploy --force
```
But in the rapid development process of web apps you will continue deploying the edited source.

As alternative after deploying for the backend functionality is to  start the Gulp web server just like for front-end development on another host:

`~moode $ node\_modules/.bin/gulp watch –build`

The Browser needs to be opened manually in this case.

If a new deploy for the backend is required ctrl-c to abort the Gulp web server and do a new build/deploy.

# 6. Known issues
## 6.1. Gulp not found

If running Gulp results in a command not found:
```
~moode $ gulp
gulp: command not found
```
The path can be specified explicitly as is shown below.
```
~/moode $ `node\_modules/.bin/`gulp -v
CLI version: 2.2.1
Local version: 4.0.2
```

Alternatively the the `node_modules/.bin/` directory can be added to the search path or Gulp can be installed global.

`~/moode $ sudo npm install -g gulp`

## 6.2. Slow Gulp web server

If the built in Web server is very slow, the issue is most likely with networks that have .local as a domain. In that case just add to your hosts file moode.test with it's IP address.

'192.168.178.152         moode.test'

And if the Gulp web server is used on the Raspberry Pi itself

`127.0.0.1         moode.test`

If the Web server is running on the Raspberry Pi you can update the hosts file with the following command.

`$ sudo sed -i.bak '${s/$/ moode.test/}' /etc/hosts'

Windows

On Windows edit the file `c:\Windows\System32\Drivers\etc\hosts` with a text editor started with Admin rights.

Mac

`TBD`

## 6.3. After deploy still old version in deploy location

Gulp optimizes the build and deploy by only updating new files. In case the file in the deploy location is newer than the source it is by default not overwritten. Use the argument `–force` to always update.

## 6.4. Error: System limit for number of file watchers reached on watch

If you get this error on the Raspberry Pi you can increase the number of files allowed using the command below

`$ sudo sysctl -w fs.inotify.max\_user\_watches=100000`


# 7. Appendix

## 7.1. Script to setup development environment

The following script sets up Gulp and prepares a development environment for a new machine.
```
#!/bin/bash
#
# Setup Gulp build environment for moOde
#

GIT\_USER\_NAME="yourgithubusername"

cd ~

# increase allowed filewatches
sudo bash -c "echo \"fs.inotify.max\_user\_watches=100000\" > /etc/sysctl.conf"
sudo sysctl -w fs.inotify.max\_user\_watches=100000

# Install packages
sudo apt-get update
sudo apt-get -y install npm
git config --global user.name $GIT\_USER\_NAME
git config --global user.email "$GIT\_USER\_NAME@users.noreply.github.com"
git config --global credential.helper 'cache --timeout=7200'
git config --global credential.helper cache

# 1. Using a fork of moode
git clone https://github.com/$GIT\_USER\_NAME/moode.git
cd moode
git remote add remote https://github.com/moode-player/moode.git
git fetch remote
git checkout develop
git pull remote develop

# 2. Alternative is to clone the repo
# git remote add remote https://github.com/moode-player/moode.git
# cd moode

npm ci
node\_modules/.bin/gulp build
node\_modules/.bin/gulp deploy --test

# create a backup of the original www root
sudo mv /var/www /var/www.org
sudo node\_modules/.bin/gulp deploy
moodeutl -r
```
