# Logging

RocketMod 5 uses ILogger service for logging.
You can use it in your plugin like this:

```csharp
public class MyPlugin : Plugin
{
    private readonly ILogger logger;
    public MyPlugin(IDependencyContainer container, ILogger logger): base(container)
    {
        this.logger = logger;
    }
    
    protected override async Task OnActivate(bool isFromReload)
    {
        logger.LogFatal("This is a fatal error"); // Fatal errors which prevents the server or plugin from working, user must take action.
        logger.LogError("This is an error"); // Errors, e.g. exceptions, might require the user to take action
        logger.LogWarning("This is a warning"); // Warnings, do es not require the user to take action
        logger.LogInformation("This is some information"); // Generic log messages to inform the user
        logger.LogDebug("This is a debug message"); // Logging debug specific stuff like variable values, states etc.
        logger.LogTrace("This is a trace message"); // Tracing API, method calls etc (generates lots of logs)
    }
}
```

!!! note
    Order of log messages level is like this:

    `(least messages)                ->               (most messages)`

    `Fatal` > `Error` > `Warning` > `Information` > `Debug` > `Trace`.

    This means that, for example, when log level is set to "Warning", only "Warning", "Error" and "Fatal" messages will be shown.
    

You can also register a custom `ILogger` (by implementing `ILogger` or inheriting `BaseLogger`/`FormattedLogger`/`FileLogger`) to capture and handle log messages.