# Migration Guide

When migrating to RocketMod 5, you should first reference the NuGet packages: Rocket.API, Rocket.Core and (if your plugin depends on Rocket.Unturned features or the Unturned dlls) Rocket.Unturned.
You will have to reference UnityEngine.dll yourself as we can not provide it because of licensing issues.

Please read [Services](https://rocketmod.guide/development-guide/core-implementation/services/) before continuing.

After that, change `<MyPlugin> : RocketPlugin<X>` part of your main plugin file to  `<MyPlugin> : Plugin`.

Now update your API usage, below you can see what has changed.

## Common API Changes

| **Old API**                                     | **New API**                             |
|-------------------------------------------------|-----------------------------------------|
| UnturnedChat                                    | PlayerManager                           |
| RocketPlugin<>                                  | Plugin                                  |
| IRocketCommand                                  | ICommand, [Command]                     |
| R.Translation                                   | ITranslationProvider                    |
| Logger.Debug / Logger.Info ...                  | [Logging](https://rocketmod.guide/development-guide/getting-started/logging/)                             |
| Any event (OnXXXX) (e.g. OnPlayerConnected)     | [Eventing](https://rocketmod.guide/development-guide/getting-started/eventing/)                            |
| IRocketPermissionsProvider                      | [Permissions](https://rocketmod.guide/development-guide/getting-started/permissions/)                         |
| R and U                                         | [Services](https://rocketmod.guide/development-guide/misc/services/)                            |
| Plugin.Configuration                            | [Configurations](https://rocketmod.guide/development-guide/getting-started/configurations/)                       |
| Update(), FixedUpdate(), etc...                 | [Scheduling](https://rocketmod.guide/development-guide/getting-started/scheduling/)                              |

## MonoBehaviours
If you want to make true universal plugins, you should avoid using MonoBehaviour under all circumstances. You can use the [TaskScheduler](https://rocketmod.guide/development-guide/core-implementation/task-scheduler/) instead. Only use MonoBehaviours if you need to add components which need access to other component data and hooks (e.g. OnCollision, Rigidbodies, etc).
