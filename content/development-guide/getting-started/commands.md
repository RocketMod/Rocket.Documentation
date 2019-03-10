# Commands

Commands are an interface for users and other plugins to trigger specific actions of your plugins. They are usually executed via chat, console, web interface or rcon.

Example command: `/kick PlayerA "Some kick Reason"`.

## Implementing Commands
There are two ways to implement commands in RocketMod 5. The first one is by implementing the ICommand interface, the second one is by using the `[Command]` attribute.

### 1. Registering Commands with ICommand
Implement the `ICommand` interface like this:
```csharp
using Rocket.API.Commands;
using Rocket.API.User;
using System.Drawing;

namespace SamplePlugin
{
    public static SampleCommand : ICommand
    {
        public string Name { get; } = "Sample";
        public string [] Aliases { get; } = null;
        public string Summary { get; } = "My sample plugins sample command.";
        public string Description { get; } = null;
        public string Syntax { get; } = "";
        public IChildCommand [] ChildCommands { get; } = null;
        
        // Allow any user type to execute this command
        public bool SupportsUser (IUser user) => true; 

        public async Task ExecuteAsync(ICommandContext context)
        {
             await context.User.SendMessageAsync("The sample command says hi!", Color.Yellow);
        }
    }
}
```
* **Name:**		The commands name (e.g. `rocket`, `buy`, etc).
* **Aliases:**		A list of alternative for the name, not required.
* **Summary:**		A short summary of the commands function.
* **Description:**	A long description entailing the commands functionality, not required.
* **Syntax:**	        A syntax string for the command `[]` = *optional* and `<>` = *required* (e.g. [steamId] <itemId>).
* **ChildCommands:**	A list of child commands (e.g. `shop sell`, `shop buy` : `buy` and `sell` are child commands of `shop`). Optional.
* **SupportsUser:**	A bool to check if a user passed through is supported for this command. For example, if you only want Console to be able to run your command, you can use `SupportsUser(IUser user) => user is IConsole`
* **ExecuteAsync:**   The method that gets invoked when someone executes your command.

`ICommand`s are automatically registered when your plugin loads. If you want to disable automatic registration, use `[DontAutoRegister]` on top of your command class.

#### Getting UnturnedPlayer from User
```csharp
public async Task ExecuteAsync (ICommandContext context)
{
    UnturnedPlayer player = ((UnturnedUser) context.User).Player;
    player.Kill();
}
```
!!! note
    Ensure that `SupportsUser` is set to `user is UnturnedUser` or add manual checks like `if(context.User is UnturnedUser)`, otherwise you might get `InvalidCastException` when a different user type (e.g. the console) calls the command.

#### Registering Child Commands
Child commands have their own relative contexts, aliases, summaries, syntaxes and permissions. For example, on `/sample joke 1`, `context.Parameters[0]` will be equals to `1`.

Child Commands can only be used for `ICommand`s or other `IChildCommand`s. Support for `[Command]` does not exist at the moment.

First create a command that inherits from IChildCommand *(it has the same structure as ICommand)*.
```csharp
public static SampleJokeCommand : IChildCommand
{
    public string Name { get; } = "Joke";
    public string[] Aliases { get; } = null;
    public string Summary { get; } = "Tells a joke.";
    public string Description { get; } = null;
    public string Syntax { get; } = "";
    public IChildCommand [] ChildCommands { get; } = null;
	
    public bool SupportsUser (IUser user) => true;
	
    public async Task ExecuteAsync(ICommandContext context)
    {
        await context.User.SendMessageAsync("<insert generic joke here>", Color.Yellow);
    }
}
```

After that, set `ChildCommands` to the freshly created class in your base command.
```csharp
public IChildCommand [] ChildCommands { get; } = new IChildCommand []
{
    new SampleJokeCommand ()
};
```

Now `/sample joke` should work. You can also use `/help sample joke` to get help about this child command.

#### Accessing Plugin Instances
Just add the following to your `SampleCommand` class.

```csharp
private MyPluginMain myPlugin;

public SampleCommand (IPlugin plugin)
{
    this.myPlugin = (MyPluginMain)plugin;
}
```
The IPlugin supplied here is automatically sdt to the plugin which registers the command (which is normally your plugin).

Now you can use `myPlugin` anywhere to access your plugin.

### 2. Registering Commands with `[Command]` Attribute
First create a new class:
```csharp
public class MyPluginCommands
{

}
```

After that, add methods with the `[ICommand]` attribute:
```csharp
public class MyPluginCommands
{
    [Command(Summary = "Kills a player.")] //By default, name is the same as the method name and it will support all users
    public async Task KillPlayer(IUser sender, IPlayer target)
    {
        if (target.IsOnline && target.Entity is ILivingEntity entity)
            await entity.KillAsync(sender);
        else // the game likely does not support killing players (e.g. Eco)
            await sender.SendMessageAsync("Target could not be killed :(");
    }

    [Command(Name = "Broadcast", Summary = "Broadcasts a message.")]
    [CommandAlias("bc")]
    [CommandUser(typeof(IConsole))] // only console can call it
    public async Task Broadcast(IUser sender, ICommandContext context)
    {
        await userManager.BroadcastAsync(sender, translations.GetAsync("broadcast", sender, context.GetArgumentLine(0));
    }
}
```

Unlike `ICommand`s, `[Command]`-based commands are not automatically registed. You can register the class above like this:
```csharp
public class MyPluginMain : Plugin
{
    public MyPluginMain(IDependencyContainer container) : base ("MyPlugin", container)
    {
			
    }

    protected override async Task OnActivate(bool isFromReload)
    {
        MyPluginCommands myCommands = new MyPluginCommands();
        RegisterCommandsFromObject(myCommands);
    }
}
```

Command parameters are automatically set according to the order they were added. There are some special parameters which are not supplied by the user and which are not part of the syntax:

* **IUser**: The user calling the command.
* **ICommandContext**: The command context.
* **ICommandParameters**: The parameters of the command.
* **IDependencyContainer**: The dependency container.

Anything else is automatically converted from what the user inputs.

For example:
```cs
[Command(Summary = "Sends a private message to a user.")]
public async Task SendMessage(IUser sender, IUser target, string[] message, ICommandContext commandContext)
{
    await target.User.SendMessageAsync($"[DM] {sender.UserName}: {string.Join(message, " ")}", Color.Green);
}
```
In this example, the command syntax would be `/sendmessage <target> <message>`.

## Command Permissions
Command permissions are determined by the active `CommandHandler` at runtime based on the `CommandContext`, so there is no way of predicting the permission that is going to be checked. Your users should use `/help <command> [subcommand] [sub-subcommand] [...]` to find out what the permission for your command is. The default `CommandHandler` provided by RocketMod uses the scheme `<pluginname>.<command>.<subcommand>...`, however any plugin can change this behaviour.

## Command Contexts
Command contexts representate the current command with its parameters, the executing user and the dependency container associated with it. 

### Getting Parameters
```csharp
public async Task ExecuteAsync(ICommandContext context)
{
    // Get ulong at index 0 (first parameter).
    // For example, if the command is /sample 4345534 asdasd, we expect "steamId" to be equal to 4345534u.

    // Method 1: this will manually throw exceptions / show error messages
    if (context.Parameters.TryGet<ulong> (0, out ulong steamId))
        throw new CommandWrongUsageException("Missing or invalid SteamID parameter.");

    // Method 2: this will automatically throw CommandWrongUsageException
    ulong steamId = context.Parameters.Get<ulong>(0);
}
```

## Best Practices

* Do not handle sub commands manually, e.g. like this:
```csharp
public async Task ExecuteAsync(ICommandContext context)
{
    string subCommand = context.Parameters.Get<string>(0);
    
    switch(subCommand.ToLower())
    {
        case "kill":
            if(await permissionProvider.CheckPermissionAsync(context.User, "kill"))
            {
                //Do something
                ...
            }
            break;

        case "something":
            //Do something
            ...
            break;
    }
}

``` 

This is an inconsistent way of handling commands and will cause issues like permissions not getting automatically assigned or manually getting assigned in a non-standard way. The `/help` command can also not provide information on any of these sub commands as it can not know these exist. It will also prevent consistent error messages when permissions do not match or the command was not found.

Instead of creating subcommands manually, create a child command as explained above.