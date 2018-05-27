> **Important: The examples in here may be outdated, feel free to help us keeping them up to date, you can simply edit them if you are logged in on GitHub.**

***
## Creating a Configuration
Configurations in RocketMod 5 works similar to how it did in the past.

### Example Configuration
```csharp
using Rocket.Core.Plugins;

namespace SamplePlugin
{
	// 'Config' can be the name of any class you want be it PluginConfig or Configuration.
	public class Main : Plugin <Config>
	{
		public Main (IDependencyContainer container) : base ("PluginName", container)
		{
			
		}
	}

	// Doesn't need to inherit from anything. Simply add in any variables you need for your config in this class.
	// The assigned vlaues will act as defaults
	public class Config
	{
		public string WelcomeMsg { get; set; } = "MyWelcomeMsg";
		[ConfigArray (ElementName = "Class")]
		public ConfigClass [] Classes { get; set; } = new ConfigClass []
		{
			new ConfigClass ()
			{
				Id = 32,
				Amount = 2
			}
		};
	}

	public class ConfigClass
	{
		public int Id { get; set; };
		public int Amount { get; set; };
	}
}
```

### Reading from Configuration
To use the values from the configuration you can use the following code
```csharp
string myMsg = ConfigurationInstance.WelcomeMsg;
```

### Saving Configuration
To save the config you can run the following
```csharp
Configuration.Save ();
```

### Creating Multiple Configurations
```csharp
using Rocket.Core.Plugins;
using Rocket.Core.Configuration;

namespace SamplePlugin
{
	// 'ConfigOne' will act as the base configuration and use ConfigurationInstance to read values
	public class Main : Plugin <ConfigOne>
	{
		private IConfiguration config;
		private ConfigTwo configTwo;
		
		protected Main (IDependencyContainer container, IConfiguration config) : base ("PluginName", container)
		{
			this.config = config;
		}
		
		protected override void OnLoad (bool isFromReload)
		{
			// Initiate second config
			ConfigurationContext ctx = new ConfigurationContext ("FileName");
			config.Load (ctx, new ConfigTwo ());
			
			// Load from file
			configTwo = config.Get<ConfigTwo> ();
			
			// Save to file
			config.Set (configTwo);
			config.Save ();
		}
	}

	// Doesn't need to inherit from anything. Simply add in any variables you need for your config in this class.
	// The assigned vlaues will act as defaults
	public class ConfigOne
	{
		public string MyWelcomeMsg { get; set; } = "MyWelcomeMsg";
		[ConfigArray (ElementName = "Class")]
		public ConfigClass [] Classes { get; set; } = new ConfigClass []
		{
			new ConfigClass ()
			{
				Id = 32,
				Amount = 2
			}
		};
	}

	public class ConfigClass
	{
		public int Id { get; set; };
		public int Amount { get; set; };
	}
	
	public class ConfigTwo
	{
		public string SampleVariable { get; set; }
	}
}
```
