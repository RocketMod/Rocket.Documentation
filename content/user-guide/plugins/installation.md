## Installing plugins on your server

## Manual Install

First start by downlading the plugin from your chosen source. After that add the `plugin-name.dll` to the `Rocket/Plugins` folder in your server. If the plugin came with libraries then add all those .dll to the `Rocket/Libraries` folder. Once thats done start your server and wait for all the plugins configuration files to generate, after its on you can shutdown and go to `Rocket/Plugins/plugin-name` and edit the `plugin-name.Configuration.xxx`.

## From Console
You can run the following command to install and manage plugins from [RocketMod Harbor](https://harbor.rocketmod.net):

### Install
Downloads and installs a plugin.
```
rocket install <plugin-name>
```

**Example:**
```
rocket install uconomy
```

### Remove
Removes a plugin.

```
rocket remove <plugin-name>
```

### Updating
Updates a plugin.

```
rocket update <plugin-name>
```