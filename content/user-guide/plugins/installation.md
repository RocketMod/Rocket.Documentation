## Installing plugins on your server

## Manual Install

First start by installing the plugin from your chosen source. Then add the `plugin-name.dll` to the `Rocket/Plugins` folder in your server. If the plugin came with libraries/packages then add all those .dll to the `Rocket/Packages` folder. Once thats done run your server and wait for all the plugins configuration files to generate, after its on you can shutdown and go to `Rocket/Plugins/plugin-name` and edit the `plugin-name.Configuration.xxx`.

## From Console

### Install
Start by turning on your server. And then running this command
```
rocket install <plugin-name>
```

**Example**
```
rocket install uconomy
```

### Uninstall
Works the same as install except this time it will remove the plugin

```
rocket uninstall <plugin-name>
```

### Updating
Updated a selected plugin

```
rocket update <plugin-name>
```

### Setting up config
This command will let you fill out certain config fields from console.
```
rocket setup <plugin-name>
```