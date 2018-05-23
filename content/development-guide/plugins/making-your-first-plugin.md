# Making your first plugin

In this guide we will cover:

1. Setting up Visual Studio
2. Creating a basic plugin

## Setting up Visual Studio
### Creating the Project
To get started with our plugin first create a new project in Visual Studio you will be greeted with a window which should look like the one below.

![alt-text](https://i.imgur.com/QyYj0Ny.png)

Make sure that the .NET framework 3.5 is selected (This could change in the future). Give your plugin a name, in this case we'll be making a Welcome Messager. And select a save location for it.

### Selecting NuGet Packages
Once your have your project creating go to 'Tools>NuGet Package Manager>Manage NuGet Packages for Solution' a new window will open, In this window go to the Browse tab and search for 'Rocket.API' and 'Rocket.Core' and install them. Once the packages are installed we can move onto making the plugin.

## Making the Plugin

Well start by renaming the pre-existing Class1.cs file to Main.cs from the solution explorer and then opening the file. Add the following to the top of the file

```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Eventing;
using Rocket.Core.Eventing;
using Rocket.Core.Player.Events;
using Rocket.Core.Plugins;
using Rocket.Core.User;
```

your file should now look something like this

```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Eventing;
using Rocket.Core.Eventing;
using Rocket.Core.Player.Events;
using Rocket.Core.Plugins;
using Rocket.Core.User;

namespace WelcomeMessager
{
	public class Main
	{
	
	}
}
```

We'll start by making Main inherit from Plugin

```csharp
public class Main : Plugin <Config>
{
	protected Main (IDependencyContainer container) : base ("Welcome Messager", container)
	{
		
	}
}
```

'Config' will show up as an error for now just ignore this. We now create a second class called Config

```csharp
public class Config
{
	public string WelcomeMsg { get; set; } = "Welcome to my Server";
}
```

This should remove the previous error warning. Any variables added to this class will act as our configuration.

Now that our configuration is setup we'll move onto making the event listener by creating a new class which will inherit the events we need

```csharp
public class EventListener : IEventListener<PlayerConnectedEvent>
{
	private Config config;

		public EventListener (Config config)
		{
			this.config = config;
		}

		[EventHandler]
		public void HandleEvent (IEventEmitter emitter, PlayerConnectedEvent @event)
		{
			@event.User.SendMessage (config.WelcomeMsg);
		}
}
```

We pass our config through the constructor so it can later be used when we handle the event, in this case our event is simply sending a welcome message to the connected user.

To finish the plugin we need to register our event to do this we go back to our Main class and add an OnLoad

```csharp
protected override void OnLoad (bool isFromReload)
{
	EventManager.AddEventListener (this, new EventListener (ConfigurationInstance));
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
	public class Main : Plugin<Config>
	{
		protected Main (IDependencyContainer container) : base ("Welcome Messager", container)
		{

		}

		protected override void OnLoad (bool isFromReload)
		{
			EventManager.AddEventListener (this, new EventListener (ConfigurationInstance));
		}
	}

	public class Config
	{
		public string WelcomeMsg { get; set; } = "Welcome to my Server";
	}

	public class EventListener : IEventListener<PlayerConnectedEvent>
	{
		private Config config;

		public EventListener (Config config)
		{
			this.config = config;
		}

		[EventHandler]
		public void HandleEvent (IEventEmitter emitter, PlayerConnectedEvent @event)
		{
			@event.User.SendMessage (config.WelcomeMsg);
		}
	}
}
```

You can now build the plugin and upload it to your testing server.
