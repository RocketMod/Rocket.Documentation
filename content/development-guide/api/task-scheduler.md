The TaskScheduler is the replacement for the Update function from RocketMod 4 while also adding improved features.

## Overview
With the TaskScheduler you can:

1. Run a method every frame (Same as Update).
2. Run a method the next frame.
3. Run a method every physics update (Same as FixedUpdate).
4. Run a method next physics update.
5. Run a method every frame on a seperate thread (Same as Update just on seperate thread).
6. Run a method the next frame on a seperate thread.

## Base
To start you first need to pass the ITaskScheduler through the constructor of your plugin.
```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Scheduler;
using Rocket.Core.Plugins;

namespace SamplePlugin
{
	public class Main : Plugin
	{
		private ITaskScheduler taskScheduler;

		public Main (IDependencyContainer container, ITaskScheduler taskScheduler) : base ("Sample Plugin", container)
		{
			this.taskScheduler = taskScheduler;
		}
	}
}
```

Once you have the task scheduler you can begin to use it. In each case you will need to pass in the plugin, if you are scheduling a task from within the class that inherits from Plugin then you can just pass `this`, otherwise you will need to pass `this` through to those other classes;

## ScheduleEveryFrame
This will schedule a method to run every [Update](https://docs.unity3d.com/ScriptReference/MonoBehaviour.Update.html) frame.

**Please read the ASync version to see if you should be using it instead.**
```csharp
protected override void OnLoad (bool isFromReload)
{
	// The SampleMethod string acts as a user friendly name can be anything you want.
	taskScheduler.ScheduleEveryFrame (this, SampleMethod, "SampleMethod");
}

private void SampleMethod ()
{
	// Do something...
}
```

## ScheduleNextFrame
This will schedule a method to run the next [Update](https://docs.unity3d.com/ScriptReference/MonoBehaviour.Update.html) frame and **it will run only once**. 

**Please read the ASync version to see if you should be using it instead.**

I'm going to skip the padding above and just focus on the taskScheduler bit.
```csharp
taskScheduler.ScheduleNextFrame (this, SampleMethod, "SampleMethod");
```

## ScheduleEveryPhysicUpdate
This will schedule a method to run every [FixedUpdate](https://docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html) frame.
```csharp
taskScheduler.ScheduleEveryPhysicUpdate (this, SampleMethod, "SampleMethod");
```

## ScheduleNextPhysicsUpdate
This will schedule a method to the next [FixedUpdate](https://docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html) frame and **it will run only once**.
```csharp
taskScheduler.ScheduleNextPhysicUpdate (this, SampleMethod, "SampleMethod");
```

## ScheduleEveryAsyncFrame
This and the next functions are probably the most imporant. This will run your method every frame on a **seperate** thread, this is vital for performance, if your method is doing some calculation and will have a delay it should be run on a seperate thread to avoid lagging the server, the next one is probably more useful then the this since its only run once but still.
```csharp
taskScheduler.ScheduleEveryAsyncFrame (this, SampleMethod, "SampleMethod");
```

## ScheduleNextAsyncFrame
This will run your method once on the next frame in a **seperate** thread.

> **Please use this for anything SQL or database related. Don't be that plugin that lags the server.**
```csharp
taskScheduler.ScheduleNextAsyncFrame (this, SampleMethod, "SampleMethod");
```

## ScheduleDelayed
This will run your method once after the set delay time and can be both on **main** thread and **seperate** thread.
It is also destroyed after reload.
```csharp
TimeSpan runAfter = TimeSpan.FromSeconds (15); // Runs after 15 seconds
taskScheduler.ScheduleDelayed (this, SampleMethod, "SampleMethod", runAfter, true); // Async
taskScheduler.ScheduleDelayed (this, SampleMethod, "SampleMethod", runAfter); // No Async
```

## ScheduleAt
This will run your method once after the set delay time and can be both on **main** thread and **seperate** thread.
It is also destroyed after reload.
```csharp
TimeSpan runAfter = TimeSpan.FromSeconds (15); // Runs every 15 seconds
taskScheduler.ScheduleAt (this, SampleMethod, "SampleMethod", runAfter, true); // Async
taskScheduler.ScheduleAt (this, SampleMethod, "SampleMethod", runAfter); // No Async
```

## SchedulePeriodically
This will run your method every set amount of time and can have a delay also can be both on **main** thread and **seperate** thread.
It is also destroyed after reload.
```csharp
TimeSpan runEvery = TimeSpan.FromSeconds (30); Run every 30 Seconds
TimeSpan runAfter = TuneSpan.FromSeconds (5); After 5 Second Delay (were applicable)
taskScheduler.SchedulePeriodically (this, SampleMethod, "SampleMethod", runEvery, null, true); // Async
taskScheduler.SchedulePeriodically (this, SampleMethod, "SampleMethod", runEvery, runAfter, true); // Delayed Async
taskScheduler.SchedulePeriodically (this, SampleMethod, "SampleMethod", runEvery); // No Async
taskScheduler.SchedulePeriodically (this, SampleMethod, "SampleMethod", runEvery, runAfter); // Delayed
```

## Extra
### Getting a List of Active Tasks

This will also return tasks scheduled by other plugins
```csharp
taskScheduler.Tasks
```

### Cancelling a Task

There are two methods of doing this

**1**. Use this method if you arent sure if the task is canceled and wanna know if it was
```csharp
ITask myTask = taskScheduler.ScheduleEveryFrame (this, SampleMethod);

// Returns a boolean.
// true 	= Canceled.
// false 	= Was already canceled.
taskScheduler.CancelTask (myTask);
```

**2**. Use this method if you just want to cancel and dont care if it was previously canceled

**Note:** This still just calls `taskScheduler.CancelTask (...);` however it might be more convenient for you to use. `myTask.Cancel ();` in some cases.
```csharp
myTask.Cancel ();
```

### Passing Arguments Through
```csharp
protected override void OnLoad (bool isFromReload)
{
	taskScheduler.ScheduleNextFrame (this, () => SampleMethod ("A little touch of wizard"), "Wizard's SampleMethod");
}

public void SampleMethod (string sampleParameter)
{
	// Do Something...
}
```

### Returning Data / Callback Function
You can't use return however you can create a callback function.
```csharp
protected override void OnLoad (bool isFromReload)
{
    taskScheduler.ScheduleNextAsyncFrame (this, SampleMethod, "SampleMethod");
}

public void SampleMethod ()
{
	object sampleData = // Do Something...
	SampleMethodCallback (sampleData);
}

public void SampleMethodCallback (object sampleReturn)
{
	// Do some more things...
}
```
