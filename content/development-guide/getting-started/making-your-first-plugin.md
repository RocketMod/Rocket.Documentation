# Making your first plugin

In this guide we will cover:

1. Setting up Visual Studio.
2. Creating a basic plugin.

## Setting up Visual Studio
Firstly download and install Visual Studio Community Edition. On the installer, you should select all .NET Core packages.

### Creating the Project
After installing and starting Visual Studio, create a new project. You will be greeted with a window which should look like the one below.

![alt-text](https://i.imgur.com/QyYj0Ny.png)

Make sure that **Class Library (.NET Standard)** is selected. Give your plugin a name, in this case we'll be making a Welcome Messager. Finally select a save location for it and click "OK".

### Installing NuGet Packages
Once you have created project, go to "Tools > NuGet Package Manager > Manage NuGet Packages for Solution". In the NuGet window go to the "Browse" tab and search for "Rocket.Core" and install it. If you are making a plugin for a specific game (e.g. Unturned), install the package for it as well (e.g. Rocket.Unturned). Once the packages are installed we can start making the plugin.

## Getting started with the plugin
We will start by renaming the pre-existing Class1.cs file to MyPluginMain.cs from the solution explorer. Add the following `using`s the top of the file:
```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Eventing;
using Rocket.Core.Eventing;
using Rocket.Core.Player.Events;
using Rocket.Core.Plugins;
using Rocket.Core.User;
```

Your file should now look something like this
```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Eventing;
using Rocket.Core.Eventing;
using Rocket.Core.Player.Events;
using Rocket.Core.Plugins;
using Rocket.Core.User;

namespace WelcomeMessager
{
    public class MyPluginMain
    {

    }
}
```

We'll start by making MyPluginMain inherit from Plugin:
```csharp
public class MyPluginMain: Plugin<MyPluginConfig>
{
    public MyPluginMain(IDependencyContainer container): base ("Welcome Messager", container)
    {
	
    }
}
```

'MyPluginConfig' will show up as an error so we now create a second class called MyPluginConfig:
```csharp
public class MyPluginConfig
{
    public string WelcomeMsg { get; set; } = "Welcome to my Server";
}
```

This should remove the previous error warning. Any variables added to this class will be configurable by the plugin user.

Now that our configuration is setup we'll add an event listener by creating a new class which will inherit the events we need:

```csharp
public class EventListener : IEventListener<PlayerConnectedEvent>
{
    private MyPluginConfig config;

    public EventListener (MyPluginConfig config)
    {
        this.config = config;
    }

    [EventHandler]
    public async Task HandleEvent (IEventEmitter emitter, PlayerConnectedEvent @event)
    {
        await @event.Player.User.SendMessageAsync(config.WelcomeMsg);
    }
}
```

We pass our config through the constructor so it can later be used when we handle the event, in this case our event is simply sending a welcome message to the connected user.

To finish the plugin we need to register our event listener. To do this we go back to our MyPluginMain class and add an OnActivate like this:

```csharp
protected override async Task OnActivate(bool isFromReload)
{
    EventBus.AddEventListener(this, new EventListener(ConfigurationInstance));
}
```

The plugin is now done and it should look something like this.

```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Eventing;
using Rocket.Core.Eventing;
using Rocket.Core.Player.Events;
using Rocket.Core.Plugins;
using Rocket.Core.User;

namespace WelcomeMessager
{
    public class MyPluginMain : Plugin<MyPluginConfig>
    {
        public MyPluginMain(IDependencyContainer container) : base ("Welcome Messager", container)
        {

        }

        protected override async Task OnActivate (bool isFromReload)
        {
            EventBus.AddEventListener (this, new EventListener (ConfigurationInstance));
        }        
    }

    public class MyPluginConfig
    {
        public string WelcomeMsg { get; set; } = "Welcome to my Server";
    }

    public class EventListener : IEventListener<PlayerConnectedEvent>
    {
        private MyPluginConfig config;

        public EventListener (MyPluginConfig config)
        {
            this.config = config;
        }

        [EventHandler]
        public async Task HandleEvent (IEventEmitter emitter, PlayerConnectedEvent @event)
        {
            await @event.Player.User.SendMessageAsync(config.WelcomeMsg);
        }
    }
}
```

You can now build the plugin and upload it to your testing server.
