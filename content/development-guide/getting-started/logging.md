RocketMod 5 uses ILogger service for logging.
User it in your plugin like this:

```cs
public class MyPlugin : Plugin
{
    public MyPlugin(IDependencyContainer container) : base(container)
    {
        
    }
    
    protected override void OnLoad (bool isFromReload)
    {
        Logger.Log ("Plugin Loaded");
    }
}
```

To register your own logger, see [Services](https://rocketmod.guide/development-guide/core-implementation/services/).
