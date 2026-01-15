# MOS Plugin Repository

This repository contains a plugin for **MOS**.  
It explains how to create and configure your own MOS plugin, including the required project structure, configuration files, optional settings, functions, and build workflow.

---

## Repository Structure

A typical MOS plugin repository is structured as follows:
```
.
├── page/
│   ├── plugin.config.js
│   ├── index.html
│   └── src/
│       └── Plugin.vue
├── functions            # Optional: bash functions executed by MOS
├── settings.json        # Optional: plugin settings
├── .github/
│   └── workflows/
│       └── release.yml  # Example GitHub Actions workflow
└── README.md
 ```

---

## Required: Plugin Page (Vue)

Every MOS plugin must provide a Vue-based configuration page or at least a basic page.

### `page/plugin.config.js`

This file defines the plugin metadata and **must be configured**:

```js
export default {
  name: "pluginname",              // Folder name used by MOS (no spaces!)
  displayName: "Plugin Name",       // Human-readable plugin name
  description: "Short plugin description",
  icon: "",                         // "" if image.png exists, or e.g. "mdi-nas"
  author: "Author Name or GitHub",
  homepage: "https://example.com"
}
```

Field explanation:

    name
      Internal plugin name and folder name on MOS.
      Must not contain spaces.

    displayName
      Display / beauty name shown in the MOS UI.

    description
      Short description of what the plugin does.

    icon
      Use "" if you provide a custom image.png  
      Or specify an MDI icon (e.g. mdi-nas)

    author
      Plugin author (real name or GitHub handle).

    homepage
      Project or plugin homepage URL.

### page/index.html

Set the browser page title for your plugin:
```
<title>DVB Driver Plugin</title>
 ```
This title is used as the page title in the browser.

## page/src/Plugin.vue

The main Vue component must be named exactly: `Plugin.vue`

This file contains the UI and frontend logic of the plugin.

## Optional: Plugin Settings

Plugins can optionally provide a settings.json file in the root of the repository.

Example:
```
{
  "driver": "libreelec"
}
```
 
## Settings API

Settings can be accessed via the MOS API: `/mos/plugins/settings/{pluginName}`

- {pluginName} must match the name field from plugin.config.js  
  Supported HTTP methods:  
  GET – retrieve settings  
  POST – update settings

## Optional: Plugin Functions

Plugins may provide a functions file containing bash functions.
Default Functions Executed by MOS

If present, MOS will automatically call these functions:

    install
      Executed when the plugin is installed.

    uninstall
      Executed when the plugin is uninstalled.

    plugin_update
      Executed when the plugin is updated.

    mos_start
      Executed when MOS starts.

    mos_osupdate
      Executed during an OS or kernel update.

## Custom Functions

You can define additional custom bash functions inside the functions file.

These functions can be executed using the MOS API.

Endpoint: `/mos/plugins/executefunction`
```
{
  "plugin": "string",
  "function": "string"
}
```
    plugin → value of name from plugin.config.js
    function → name of the function inside the functions file

## Executing Binaries

Plugins can execute binaries using the MOS API endpoint: `/mos/plugins/query`

### Execution Rules

- Only binaries located in `/usr/bin/plugins/` are allowed to be executed
- The plugin must provide the binary itself
- Symlinks to other binaries are allowed
- Executing binaries from any other location is not permitted

---

### API Request

To execute a binary and receive its output, send a request with the following JSON body:
```
{
"command": "nvidia-smi",
"args": [
  "-q",
  "-x"
],
"timeout": 5,
"parse_json": false
}
```

Field Description

    command
      Name of the executable located in /usr/bin/plugins/

    args
      Array of command-line arguments passed to the executable

    timeout
      Maximum execution time in seconds before the command is terminated

    parse_json  
      true → MOS tries to parse the command output as JSON  
      false → raw output is returned

## Notes

    This endpoint can be used to execute CLI tools as well as full applications  
    Output is returned directly by the API  
    Proper timeout values should always be set to avoid blocking behavior

## Driver Information Endpoint

Plugins that provide kernel drivers can query driver-related information using the following endpoint.

**This endpoint is **only available** if the plugin template is configured with:**

```
driver: true
```
inside the plugin template configuration.

Endpoint `GET /mos/plugins/driver/{pluginName}`

    pluginName must match the name field from plugin.config.js

## Description

This endpoint returns information about the currently available or installed driver package for the given plugin and kernel version.

It is typically used by driver plugins to:

- Detect the active kernel version
- Locate the correct driver package
- Determine installation paths

## Response Example
```
{
  "plugin": "dvb-drivers",
  "kernel": "6.18.5-mos",
  "package": "dvb-digital-devices_20251201-1+mos_amd64.deb",
  "path": "/boot/optional/drivers/dvb-drivers/6.18.5-mos/dvb-digital-devices_20251201-1+mos_amd64.deb",
  "directory": "/boot/optional/drivers/dvb-drivers/6.18.5-mos"
}
```

## GitHub Actions & Releases

An example GitHub Actions workflow is provided in: `.github/workflows/build-plugin.yml`
 
 

This workflow:

    Builds the plugin
    Packages it
    Uploads it as a GitHub Release

## Required GitHub Repository Settings

To allow the workflow to create releases:

     Open Repository Settings
     Go to Actions → General
     Under Workflow permissions, select:
     Read and write permissions
     Click Save

**This step is required, otherwise the workflow will fail.**
License


## Notes

    The plugin name must always match the folder name used by MOS
    The Vue page must always be located inside the page/ directory
    Plugin.vue is mandatory and must not be renamed
