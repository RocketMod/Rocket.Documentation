# Eventing
Events are used to trigger actions when something happens (e.g. player joined, server is shutting down, etc.) 

The default C# events used with old RocketMod 4 were very limited. It was not possible to have prioritized events. The new API allows controlling lifespan and execution order of events.

## Listening for events
To listen to events, you will have to create a class which implements `IEventListener<YourTargetEvent>`. 
Don't forget to add `[EventHandler]` attribute to your method to set various options like priority.

You have to register your event listener with `EventBus.AddEventListener(YourPlugin, YourIListener)` on plugin load to register your events. 

### Example listener class

```csharp
public class MyEventListener : IEventListener<PlayerConnectedEvent>, IEventListener<PlayerChatEvent>
{
    private IChatManager chatManager;
    public MyEventListener(IChatManager chatManager)
    {
        this.chatManager = chatManager;
    }

    [EventHandler]
    public async Task HandleEvent(IEventEmitter emitter, PlayerConnectedEvent @event)
    {
        await chatManager.BroadcastAsync(@event.Player.Name + " joined");
    }

    [EventHandler]
    public async Task HandleEvent(IEventEmitter emitter, PlayerChatEvent @event)
    { 
        if(ContainsBadWord(@event.Message))
        {
            @event.IsCancelled = true;
        }
    }
    
    public bool ContainsBadWord(string message)
            => return message.Contains("Trojaner");
}

public class MyPluginMain : Plugin
{
    private readonly IChatManager chatManager;
    private readonly IEventBus eventBus;

    public MyPluginMain(IDependencyContainer container, IChatManager chatManager, IEventBus eventBus) : base("MyPlugin", container)
    {
        this.chatManager = chatManager;
        this.eventBus = eventBus;
    }

    protected override async Task OnActivate(bool isFromReload)
    {
        eventBus.AddEventListener(this, new MyEventListener(chatManager));
    }
}
```

!!! note
    Execution order of events is like this:   


    `Lowest` -> `Low` -> `Normal` -> `High` -> `Highest` -> `Monitor`.


    You should only use monitor when it does not impact anything, for example when you use it purely for logging purposes.

### Event priorities and cancellation
EventHandlers can have priorities by using `[EventHandler(Priority = EventPriority.X)]`. 

The method with `Lowest` will be executed first, and `Monitor` will be executed last. `Monitor` should be only be used by listener which do not do any changes.
If any event listener decides to cancel the event before your event listener is called, your listener will not be called.
You can use `[EventHandler(IgnoreCancelled = true)]` to receive cancelled events. This also allows you to un-cancel events by setting `Event.IsCancelled` to false.

## Creating custom events (e.g. for other plugins)
Create a new class which extends `Rocket.API.Eventing.Event`. If you want to have it cancellable, add `ICancellableEvent` to it. 

```csharp
public class MyEvent: Event, ICancellableEvent
{
        public string SomeData { get; set; }

        public MyEvent() : this(true)
        {

        }

        public MyEvent(bool global = true) : base(global)
        {
        }

        public MyEvent(EventExecutionTargetContext executionTarget = EventExecutionTargetContext.Sync, bool global = true) : base(executionTarget, global)
        {
        }

        public MyEvent(string name = null, EventExecutionTargetContext executionTarget = EventExecutionTargetContext.Sync, bool global = true) : base(name, executionTarget, global)
        {
        }

        public bool IsCancelled { get; set; }
}
```

You can trigger and handle your custom events like this:

```csharp
MyEvent @event = new MyEvent();
@event.SomeData = data; 
eventBus.Emit(plugin, @event, callback: (e) => {
       //event has finished and all listeners were called
       
       if(@event.IsCancelled) //if your event extends ICancellableEvent
          return;

       // do something with @event.SomeData        
}); // plugins listening to events can change "SomeData"
```
