The TaskScheduler is the replacement for the Update function from RocketMod 4 while also adding improved features.

## Overview
With the TaskScheduler you can:

1. Run a method every frame (Same as Update)
2. Run a method the next frame
3. Run a method every physics update (Same as FixedUpdate)
4. Run a method next physics update
5. Run a method every frame on a seperate thread (Same as Update just on seperate thread)
6. Run a method the next frame on a seperate thread

## Base
To start you first need to pass the ITaskScheduler through the constructor of your plugin

```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Scheduler;
using Rocket.Core.Plugins;

namespace SamplePlugin
{
	public class Main : Plugin
	{
		private ITaskScheduler taskScheduler;

		protected Main (IDependencyContainer container, ITaskScheduler taskScheduler) : base ("Sample Plugin", container)
		{
			this.taskScheduler = taskScheduler;
		}
	}
}
```

Once you have the task scheduler you can begin to use it. In each case you will need to pass in the plugin, if you are scheduling a task from within the class that inherits from Plugin then you can just pass `this`, otherwise you will need to pass `this` through to those other classes;

## ScheduleEveryFrame
This will schedule a method to run every [Update](https://docs.unity3d.com/ScriptReference/MonoBehaviour.Update.html) frame

**Please read the ASync version to see if you should be using it instead.**

```csharp
protected override void OnLoad (bool isFromReload)
{
	taskScheduler.ScheduleEveryFrame (this, SampleMethod);
}

private void SampleMethod ()
{
	// Do something...
}
```

## ScheduleNextFrame
This will schedule a method to run the next [Update](https://docs.unity3d.com/ScriptReference/MonoBehaviour.Update.html) frame and **it will run only once**. 

**Please read the ASync version to see if you should be using it instead.**

Im gonna skip the padding above and just focus on the taskScheduler bit.

```csharp
taskScheduler.ScheduleNextFrame (this, SampleMethod);
```

## ScheduleEveryPhysicUpdate
This will schedule a method to run every [FixedUpdate](https://docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html) frame.

```csharp
taskScheduler.ScheduleEveryPhysicUpdate (this, SampleMethod);
```

## ScheduleNextPhysicsUpdate
This will schedule a method to the next [FixedUpdate](https://docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html) frame and **it will run only once**.

```csharp
taskScheduler.ScheduleNextPhysicUpdate (this, SampleMethod);
```

## ScheduleEveryAsyncFrame
This and the next functions are probably the most imporant. This will run your method every frame on a **seperate** thread, this is vital for performance, if your method is doing some calculation and will have a delay it should be run on a seperate thread to avoid lagging the server, the next one is probably more useful then the this since its only run once but still.

```csharp
taskScheduler.ScheduleEveryAsyncFrame (this, SampleMethod);
```

## ScheduleNextAsyncFrame
This will run your method once on the next frame in a **seperate** thread.

> **Please use this for anything sql or database related, don't be that plugin that lags the server.**

```csharp
taskScheduler.ScheduleNextAsyncFrame (this, SampleMethod);
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

// returns a boolean
// true 	= Canceled
// false 	= Was already canceled
taskScheduler.CancelTask (myTask);
```

**2**. Use this method if you just want to cancel and dont care if it was previously canceled

**Note**: This still just calls `taskScheduler.CancelTask (...);` however it might be more convenient for you to use `myTask.Cancel ();` in some cases.
```csharp
myTask.Cancel ();
```

### Passing Arguments Through

```csharp
protected override void OnLoad (bool isFromReload)
{
	taskScheduler.ScheduleNextFrame (this, () => SampleMethod ("A little touch of wizard"));
}

public void SampleMethod (string sampleParameter)
{
	// Do Something...
}
```

### Returning Data / Callback Function
You cant use return however you can create a callback function.

```csharp
protected override void OnLoad (bool isFromReload)
{
    taskScheduler.ScheduleNextAsyncFrame (this, SampleMethod);
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
