With RocketMod it is very easy to implement your own game:
First import Rocket.API, Rocket.Core, Rocket.Runtime.
If you use UnityEngine or UnrealEngine, make sure that you also import Rocket.UnityEngine or Rocket.UnrealEngine.

!!! note
    If you use Rocket.UnityEngine, you have to download and reference UnityEngine.dll and UnityEngine.CoreModule.dll yourself. Due licensing reasons, we can not provide engine files.

The following services are expected to be implemented (see [[Services]] for registering services using IDependencyRegistrator):
* ITaskScheduler //This is not needed if you use UnityEngine or UnrealEngine packages

* IPlayerManager

* IHost


The following events **must** be implemented:

* ImplementationReadyEvent

When your assembly loads, call `Runtime.Bootstrap()`. If everything works, your `IHost.Load()` will be called.
You need to load plugins from each `IPluginManager` at this point using `IPluginManager.Init()`.

You can visist [Rocket.Unturned](https://github.com/RocketMod/Rocket.Unturned) for an example implementation.
