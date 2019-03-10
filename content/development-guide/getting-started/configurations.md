# Configurations

Configurations are used to read, write and store variables. This allows an user to configure your plugin.

```csharp
using Rocket.Core.Plugins;

namespace SamplePlugin
{
    // 'Config' can be the name of any class you want be it PluginConfig or Configuration.
    public class MyPluginMain: Plugin <Config>
    {
        public MyPluginMain(IDependencyContainer container) : base ("MyPlugin", container)
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
To save the config, you can use the following method:
```csharp
await Configuration.SaveAsync();
```

### Creating Multiple Configurations
```csharp
using Rocket.Core.Plugins;
using Rocket.Core.Configuration;

namespace SamplePlugin
{
    // 'ConfigOne' will act as the base configuration and use ConfigurationInstance to read values
    public class MyPluginMain : Plugin <ConfigOne>
    {
        private IConfiguration config;
        private ConfigTwo configTwo;
		
        public MyPluginMain (IDependencyContainer container, IConfiguration config) : base ("MyPlugin", container)
        {
            this.config = config;
        }
		
        protected override async Task OnActivate(bool isFromReload)
        {
            // Initiate second config
            ConfigurationContext ctx = new ConfigurationContext ("FileName");
            await config.LoadAsync(ctx, new ConfigTwo ());
            
            // Load from file
            configTwo = config.Get<ConfigTwo> ();
			
            // Save to file
            config.Set (configTwo);
            await config.SaveAsync();
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
