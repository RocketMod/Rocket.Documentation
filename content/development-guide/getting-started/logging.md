RocketMod 5 uses ILogger service for logging.
You can use it in your plugin like this:

```cs
public class MyPlugin : Plugin
{
    private readonly ILogger logger;
    public MyPlugin(IDependencyContainer container, ILogger logger): base(container)
    {
        this.logger = logger;
    }
    
    protected override async Task OnActivate(bool isFromReload)
    {
        logger.LogInformation("This is some information");
        logger.LogWarning("This is a warning");
        logger.LogError("This is an error");
        logger.LogFatal("This is a fatal error");
        logger.LogDebug("This is a debug message");
        logger.LogFatal("This is a trace message");
    }
}
```

You can also register a custom `ILogger` (by implementing `ILogger` or inheriting `BaseLogger`/`FormattedLogger`/`FileLogger`) to capture and handle log messages.