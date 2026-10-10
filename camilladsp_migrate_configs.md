<!-- Add links to top -->

Migrating CamillaDSP Configs <!-- omit in toc -->
===============================================

(C) Tim Curtis 2026  
(C) @bitlab 2026

The source for this information is [BITLAB'S BYTES by @bitlab](https://www.bitlab.nl/page_id=2016)

## Introduction

When you use CamillaDSP configuration files from an older version, or update CamillaDSP to a new major version, you may need to update your configuration files to make them compatible with the new version.

This document describes the steps required to update the configuration files using `Files` tab in the CamillaDSP Pipeline Editor (CamillaGUI).

## Migration steps

### 1. Invalid config files

When a configuration file is invalid, you may see a message in the WebUI indicating that the output device could not be opened when attempting playback.

IMAGE

You can check whether a configuration is valid by selecting the configuration in the `File Management` section of the CamillaDSP Config screen. If the configuration is invalid, a red `X` is shown along with the reason.

IMAGE

### 2. Updating config files

Updating a configuration file is done in the Pipeline Editor by importing it into a new blank file and then saving the file. During the import all outdated elements in the file are updated.

#### Open the Pipeline Editor

- Navigate to the CamillaDSP Config screen
- Scroll down to the Pipeline Editor section
- Turn on Enable
- Turn on Expert mode
- Click OPEN

IMAGE

#### Navigate to the `Files` tab

Invalid config files will have a red `X` in the column named `Valid`. The process for updating them is described below and involves importing a given invalid config into a new blank config then naming and saving it.

IMAGE

- Click the `New blank config` button
- Click the `Import config` button
- Click the config file to be imported
- Select what to import, the top checkbox selects all items
- Click the `Import` button
- Click the `X` to close the Import screen

Scroll to the end of the Configs list and locate the `Save` icon which is to the left of the input field containing `New config.yml`.

IMAGE

- Enter a name in the input field. It can be the same as an existing config file in which case that file will be replaced by the updated file.

- Click the `Save` icon

The updated file will appear in the Configs list with a green check.

IMAGE
