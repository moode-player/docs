moOde development with Gulp

** 


**moOde development with gulp**





















**Table of Contents**

[Introduction](#_gjdgxs)	**3**

[TL;DR](#_30j0zll)	**4**

[Back and front end stuff](#_1fob9te)	4

[You do front-end stuff](#_3znysh7)	4

[Gulp](#_2et92p0)	**5**

[Features of gulp with moOde](#_tyjcwt)	5

[Relation with non gulp workflow](#_3dy6vkm)	5

[Configuration](#_1t3h5sf)	5

[About tasks](#_4d34og8)	6

[The build and deploy process](#_2s8eyo1)	7

[Complete available task list](#_17dp8vu)	8

[Temporary files](#_3rdcrjn)	9

[Prepare development host](#_26in1rg)	**11**

[Install required tooling:](#_lnxbz9)	11

[Checkout out work git repo](#_35nkun2)	11

[Setup the gulp dependencies](#_1ksv4uv)	11

[Scenarios](#_44sinio)	**12**

[Prepare the build of a release on the Raspberry Pi](#_2jxsxqh)	12

[Front-end development on development host](#_z337ya)	13

[Front-end and back-end development on the Raspberry](#_3j2qqm3)	14

[Known issues](#_1y810tw)	**16**

[Gulp not found](#_4i7ojhp)	16

[Slow gulp web server](#_2xcytpi)	16

[After deploy still old version in deploy location](#_1ci93xb)	17

[Error  ‘System limit for number of file watchers reached’ on watch](#_2bn6wsx)	17

[Appendix](#_qsh70q)	**18**

[Script to setup development environment](#_3as4poj)	18

# **
1  # **Introduction**
[Gulp](https://gulpjs.com/) with moOde is used to automate and make development easier, with the following use cases in mind:

1. Front-end and back-end development on the Raspberry Pi.
1. Prepare the build of a release on the Raspberry Pi.
1. Front-end development on development host (as Windows, Mac and Ubuntu) than the Raspberry Pi.

#

# **
1  # **TL;DR**
## **Back and front end stuff**
Assumes:

- You haved Raspberry Pi with moOde installed (7.0.0b2 or higher)
- You are logged in with a ssh client to the pi

You do:

~ $ cd ~

~ $ sudo apt-get update

~ $ sudo apt-get -y install npm

~ $ git clone https://github.com/moode-player/moode.git

~ $ cd moode

~ /moode $ npm ci

~ /moode node\_modules/.bin/gulp build

~ /moode node\_modules/.bin/gulp deploy --test

~ /moode sudo mv /var/www /var/www.org

~ /moode sudo node\_modules/.bin/gulp deploy

~ /moode moodeutl -r

Result:

- moOde front-end running with bundled and minified sources from /var/www.

## **You do front-end stuff**
Assumes:

- You haved Raspberry Pi with moOde installed (7.0.0b2 or higher) and running
- You can do front-end work on either your workstation or the Raspberry Pi, your choice.
- You read issue 6.2 slow browser and made sure moode.test points to the correct ip.

You do:

~ $ cd ~

~ $ sudo apt-get update

~ $ sudo apt-get -y install npm

~ $ git clone https://github.com/moode-player/moode.git

~ $ cd moode

~/moode $ npm ci

~/moode $ node\_modules/.bin/gulp watch --build

Result:

- You can test moOde build with bundles on your workstation at http://localhost:3000 or Pi at http://moode:3000.
- The moment you make a front-end source change, the files will be rebuild.

Oh and don't forget to read about issue 6.2 slow browser.
1  # **Gulp**
   1  ## **Features of gulp with moOde**
Gulp itself doesn’t do a thing for us. The gulp project file ~/moode/gulpfile.js contains the logic and targets of what to do. You can compare it to something like Make files.

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
  1  ## **Relation with non gulp workflow**
If the gulp workflow isn’t used, it will not sit in the way of the non gulp workflow\*. You can still use the gulp workflow for development (and check-in the source changes) and do the final deployment in the old way. 

\*) That means no automatic minifying and no use of bundling. Plain copy of the ~/moode/www is used as deployment to /var/www/.
1  ## **Configuration**
The ~moode/package.json contains the configuration of the gulp build options. 

Normally you don’t need to touch this file, only the version number will change.

The defaults for the used paths are.

|Setting|Description|Default values|
| :- | :- | :- |
|app.src |Source location|www|
|app.cache|Cache location|build/cache|
|app.dest|Build output|build/develop|
|app.dist|Temporary location for test deploy|build/distr/var/www|
|app.deploy|Actual deploy location|*/var/www*|

1  ## **About tasks**
Letting gulp do some work is done by executing gulp and providing a task to execute.

Running a task requires providing the name and optional arguments.

~/moode $ node\_modules/.bin/gulp <task\_name> [optional\_arguments]

As output it will so which tasks are executed.

Main build targets:

|Task|Description|
| :- | :- |
|gulp clean [--all] |Empties the build/build and build/distr directory, with the --all also the cache is flushed.|
|gulp build [--force] |Build frontend from {app.src} and put output in {app.dest}|
|gulp watch [--build] [--force]|<p>Start web server with as root {app.src} with proxy to moode, used for local web development.</p><p>` `When --build is given, also perform a build and use {app.dest} as web root.</p>|
|gulp deploy [--test] [--all]|<p>` `Deploys everything needed (inc php etc) {app.deploy}.</p><p>With the option --test deploy to build/dist (app.dist).</p><p>` `When used to real don't forget to sudo first and do a moodeutl -r afterwards.</p><p>This not only includes the javascript, html and css stuff, but everything in the /var/www. Including patched header and footer.</p><p>With the --all option also moodeutl and stuff in the var/local/www is deployed. You db is left intact.</p><p>With deploy files are created or updated, not deleted.</p>|


Default most tasks only update/use files that are changed (= is newer).  With force the files are always updated.


1  ## **The build and deploy process**

The following source are change on the final deploy:

- header.php
- footer.php
- index.tpl.html

Also the header.php plays an important role to detect which resources should be processed. To the header file additional comments with markup instructions are added.

For example:

<!-- build:css css/styles.min.css -->

`    `<link href="css/bootstrap.min.css" rel="stylesheet">

`    `<link href="css/bootstrap-select.min.css" rel="stylesheet">

`    `<link href="css/flat-ui.min.css" rel="stylesheet">

`    `<link href="css/jquery.pnotify.default.min.css" rel="stylesheet">

`    `<link href="css/fontawesome-moode.min.css" rel="stylesheet">

`    `<link href="css/panels.min.css" rel="stylesheet">

`    `<link href="css/moode.min.css" rel="stylesheet">

` `<!-- endbuild -->

This instructs that the contained resources are minified with source maps and are replaced by the bundle file styles.min.css.

This reduces the resources for html and css to:

Other markup instructions comment, remove or uncomment code blocks.

There are also index.html files for use in combination with the gulp web server:

- ~moode/www/index.html – includes individual resources
- ~moode/build/develop/index.html – includes resource bundles
  1  ## **Complete available task list**
You need only a small number of tasks, but for the overview lets see which tasks are present:

~/moode $ node\_modules/.bin/gulp --tasks

[09:55:14] Tasks for ~/moode/gulpfile.js

[09:55:14] ├── listplugins

[09:55:14] ├── sass

[09:55:14] ├── browserSync

[09:55:14] ├── browserReload

[09:55:14] ├── cache

[09:55:14] ├── maps

[09:55:14] ├─┬ bundle

[09:55:14] │ └─┬ <series>

[09:55:14] │   ├── cache

[09:55:14] │   ├── maps

[09:55:14] │   └── <anonymous>

[09:55:14] ├── genindexdev

[09:55:14] ├── genindex

[09:55:14] ├── patchheader

[09:55:14] ├── patchfooter

[09:55:14] ├── minifyhtml

[09:55:14] ├── artwork

[09:55:14] ├── clean

[09:55:14] ├─┬ build

[09:55:14] │ └─┬ <series>

[09:55:14] │   ├── sass

[09:55:14] │   ├── genindexdev

[09:55:14] │   ├── genindex

[09:55:14] │   ├─┬ bundle

[09:55:14] │   │ └─┬ <series>

[09:55:14] │   │   ├── cache

[09:55:14] │   │   ├── maps

[09:55:14] │   │   └── <anonymous>

[09:55:14] │   ├── artwork

[09:55:14] │   └── <anonymous>

[09:55:14] ├─┬ deployback

[09:55:14] │ └─┬ <series>

[09:55:14] │   ├── patchheader

[09:55:14] │   ├── patchfooter

[09:55:14] │   ├── minifyhtml

[09:55:14] │   └── <anonymous>

[09:55:14] ├── deployfront

[09:55:14] ├─┬ deploy

[09:55:14] │ └─┬ <series>

[09:55:14] │   ├── deployfront

[09:55:14] │   ├─┬ deployback

[09:55:14] │   │ └─┬ <series>

[09:55:14] │   │   ├── patchheader

[09:55:14] │   │   ├── patchfooter

[09:55:14] │   │   ├── minifyhtml

[09:55:14] │   │   └── <anonymous>

[09:55:14] │   └── <anonymous>

[09:55:14] └─┬ watch

[09:55:14]   └─┬ <series>

[09:55:14]     ├── sass

[09:55:14]     ├── genindexdev

[09:55:14]     ├── browserSync

[09:55:14]     └── <anonymous>

1  ## **Temporary files**
During running tasks a directory called ~/moode/build is created.

It contains 3 important main directories:

- cache - A cache of the minified files including the map files.
- develop – Output of a build. Contains bundles and also files to run the proxy web server.
- distr – Output of a test deploy 

Full overview of the content of the build directory.

~/moode $ tree -d build

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

└── distr

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



1  # **Prepare development host**
   1  ## **Install required tooling:**
Before you can use gulp we need to set up gulp. Like most modern webtooling Gulp is build with javascript and requires installation of npm and node.

For the raspberry pi and other flavors of Linux that support apt-get:

~ $ sudo apt-get install npm

For other platforms see <https://nodejs.org/en/download/>.

1  ## **Checkout out work git repo**
Probably you will use your own repo here, but for demonstration of the gulp tooling this ok.

~ $ git clone <https://github.com/moode-player/moode.git>  

For the rest of the document we assume that the checkout path is ~/moode.

1  ## **Setup the gulp dependencies**
The tools required for using gulp are installed as self containing tools in ~/moode/node\_modules. Instead of installing modules with help of the package.json , package-lock.json is used. This ensures that also the deps of deps etc are frozen to a certain version.

With the folling command those modules are installed.

~/moode $ npm ci

This will take some time. The node\_modules directory is excluded in the .gitignore file so it will not bother you during your development. The node\_modules doesn’t need to be reinstalled if you switch branches.

Because gulp itself is also installed as a package, running gulp requires a path to its binary. Let’s check the installation is successful:

~/moode $ node\_modules/.bin/gulp -v

CLI version: 2.2.1

Local version: 4.0.2

If providing the path bothers you that much, you can install gulp global or add the node\_modules/.bin to your search path.

# **
1  # **Scenarios**
In this chapter we take a look at several different uses of gulp. 

The basic idea is that you always work with the checkout source and build/deploy the changed files.

I prefer using an IDE like Visual Code (runs on Mac, Windows and Linux) which make it possible to also directly edit files, commit and run commands on a remote target as Raspberry Pi.

1  ## **Prepare the build of a release on the Raspberry Pi**


Here we bring our local branch in sync and do a deploy to the /var/www to validate the final release. We do a force to make sure newer files in the destination are overwritten even if newer.

~moode $ git fetch; git pull
~moode $ node\_modules/.bin/gulp build
~moode $ sudo node\_modules/.bin/gulp deploy --force
~moode $ moodeutl -r

Optional we could have done a test deploy before the real deploy:

~moode $ sudo node\_modules/.bin/gulp deploy –test

You can inspect the output in build/distr.
1  ## **Front-end development on development host** 

Here we assume we are working on our workstation (host as in Windows, Mac and Ubuntu) instead of on the target device Raspberry Pi (target).

The target should be running and contain a valid matching backend code.

The workflow is like:

- Start with gulp a local development web server with proxy backend calls to the target
- Editor source in your favorite editor/ide
- Save the file
- The gulp see the change an triggers a rebuild
- The browser is automatically reloaded (and your css change is visible)

The steps with gulp:

~moode $ git fetch; git pull
~moode $ node\_modules/.bin/gulp watch –build

With the –build you will use the build with bundles files else without bundles.

Add the end of the gulp output you will see something like:

`	`[Browsersync] Access URLs:

` `----------------------------------

`       `Local: http://localhost:3000

`    `External: http://0.0.0.0:3000

` `----------------------------------

`          `UI: http://localhost:3001

` `UI External: http://localhost:3001

` `----------------------------------

[Browsersync] Serving files from: build/develop

Which indicates that the development web server can be reached at [http://localhost:3000](http://localhost:3000/).

Default it will try to open a browser for you.

An important question is unanswered: how does the development web server proxy know how the raspberry pi is reachable?

It assumes that your raspberry is reachable as [http://moode.test](http://moode.test/). (see package.json). You can change that setting in the package.json. 

1  ## **Front-end and back-end development on the Raspberry**


In this case you are not only doing heavy front-end work, but also back-end stuff (= php and configuration pages).

This will require working on the target device. 

Every time you make a change to a backend file that needs to deployed for test you do:

First time:

~moode $ node\_modules/.bin/gulp build
~moode $ sudo node\_modules/.bin/gulp deploy --force
~moode $ moodeutl -r

Next time:

~moode $ sudo node\_modules/.bin/gulp deploy
~moode $ moodeutl -r

For frontend changes you could also do something like:

~moode $ node\_modules/.bin/gulp build
~moode $ sudo node\_modules/.bin/gulp deploy --force

But in the rapid development process of web apps you will continue deploying the edit source.

As alternative we can after deploying (for the backend functionality), start the gulp web server just like we did for front-end development on another host:

~moode $ node\_modules/.bin/gulp watch –build

Only we need to open a browser yourself.

If a new deploy for the backend is required ctrl-c to abort the gulp web server and do a new build/deploy.


# **
1  # **Known issues**
   1  ## **Gulp not found**
If running gulp results in a command not found:

~moode $ gulp

gulp: command not found

Because gulp itself is also installed as a package, running gulp requires a path to its binary. Let’s check if this works:

~/moode $ node\_modules/.bin/gulp -v

CLI version: 2.2.1

Local version: 4.0.2

Alternative you can add this directory to your path or install you can install gulp global:

~/moode $ sudo npm install -g gulp

1  ## **Slow gulp web server**
If the built in webserver is very slow, you're hitting a known issue with local networks that have .local as a domain. In that case just add to your hosts file moode.test with the specific ip number.

192.168.178.152         moode.test 

And if the gulp web server is used on the Pi itself

127.0.0.1         moode.test

Pi:

If the watch webserver is running on the pi you can add the moode.test with the following command.

$ sudo sed -i.bak '${s/$/ moode.test/}' /etc/hosts

Windows:

On Windows you need to edit the file c:\Windows\System32\Drivers\etc\hosts with and text editor started with Admin rights.

Mac:

1  ## **After deploy still old version in deploy location**
Gulp optimizes the build and deploy by only updating new files.

In case the file in the deploy location is newer than the source it is default not overwritten. Use the argument –force to always update.

1  ## **Error  ‘System limit for number of file watchers reached’ on watch**
If you get this error on the Raspberry Pi you can increase the number of files allowed with:

$ sudo sysctl -w fs.inotify.max\_user\_watches=100000


1  **Appendix**

   ============
   1  ## **Script to setup development environment**
The following script setup gulp and prepare a development environment for a new machine.

#!/bin/bash

\#

\# Setup intial buildenv for moOde

\#

GIT\_USER\_NAME="yourgithubusername"

cd ~

\# increase allowed filewatches

sudo bash -c "echo \"fs.inotify.max\_user\_watches=100000\" > /etc/sysctl.conf"

sudo sysctl -w fs.inotify.max\_user\_watches=100000

sudo apt-get update

sudo apt-get -y install npm

git config --global user.name $GIT\_USER\_NAME

git config --global user.email "$GIT\_USER\_NAME@users.noreply.github.com"

git config --global credential.helper 'cache --timeout=7200'

git config --global credential.helper cache

\# 1. using your own fork of moode

git clone https://github.com/$GIT\_USER\_NAME/moode.git

cd moode

git remote add remote https://github.com/moode-player/moode.git

git fetch remote

git checkout develop

git pull remote develop

\# 2. alternative if you don’t have an fork, you can use the main repo

\# git remote add remote https://github.com/moode-player/moode.git

\# cd moode

npm ci

node\_modules/.bin/gulp build

node\_modules/.bin/gulp deploy --test

\# create a backup of the original www root

sudo mv /var/www /var/www.org

sudo node\_modules/.bin/gulp deploy

moodeutl -r


version 0.3 		19

