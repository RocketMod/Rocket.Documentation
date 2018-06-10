## Installing plugins on your server

### Manual Install

First start by installing the plugin from your chosen source. Then add the `plugin-name.dll` to the `Rocket/Plugins` folder in your server. If the plugin came with libraries/packages then add all those .dll to the `Rocket/Packages` folder. Once thats done run your server and wait for all the plugins configuration files to generate, after its on you can shutdown and go to `Rocket/Plugins/plugin-name` and edit the `plugin-name.Configuration.xxx`.

### Automatic Install

Start by turning on your server. If the plugin is a universal plugin then run the following command
```
rocket install universal <plugin-name>
```
If the plugin is game specific then run the following
```
rocket install <name-of-game> <plugin-name>
```

**Example**
```
rocket install universal uconomy
```