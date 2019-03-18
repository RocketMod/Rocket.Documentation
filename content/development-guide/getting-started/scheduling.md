# Scheduling

RocketMod 5 includes a `ITaskScheduler` interface which allows to control tasks in a consistent way.

## Overview
With the TaskScheduler you can schedule a task to be run on:

1. frame updates (same as Update in Unity).

2. physics updates (same as FixedUpdate in Unity).

3. seperate thread frame updates.


Tasks are automatically destroyed on reloads and must be registered again in your plugins `OnActivate()` method.

## Getting Started
To start you first need to pass the ITaskScheduler through the constructor of your plugin.
```csharp
using Rocket.API.DependencyInjection;
using Rocket.API.Scheduler;
using Rocket.Core.Plugins;

namespace SamplePlugin
{
	public class SamplePlugin : Plugin
	{
		private ITaskScheduler taskScheduler;

		public SamplePlugin(IDependencyContainer container, ITaskScheduler taskScheduler): base("Sample Plugin", container)
		{
			this.taskScheduler = taskScheduler;
		}
	}
}
```

Once you have done that, you can begin to schedule your tasks.

### ScheduleEveryFrame
Schedules a task to run on every frame update.

```csharp
protected override async Task OnActivate(bool isFromReload)
{
	// The "My custom Task" parameter is the description for your task. It
	// should be something that can identify your plugin and the exact task. 
	// It will be shown to the user if an error occurs or when the user manages
	// tasks manually.
	taskScheduler.ScheduleEveryFrame(this, SampleTask, "My custom Task");
}

private void SampleTask ()
{
	// Do something...
}
```

### ScheduleNextFrame
Schedules a task to run on the next frame update and will run only once. This is useful if you need to access Game or Engine APIs which are not thread safe.

```csharp
taskScheduler.ScheduleNextFrame(this, SampleTask, "My custom Task");
```

### ScheduleEveryPhysicUpdate
Schedules a task to run on every physics update frame. 
```csharp
taskScheduler.ScheduleEveryPhysicUpdate(this, SampleTask, "My custom Task");
```

> **Note**: Physics update frame definition depends on the engine and game used. For Unity, it is the equivalent of "FixedUpdate". You should use Physic Update when you want to apply physics, e.g. when applying force to an object.  

### ScheduleNextPhysicsUpdate
Schedules a task to run on the next physics update frame and will run only once.
```csharp
taskScheduler.ScheduleNextPhysicUpdate(this, SampleTask, "My custom Task");
```
> **Note**: Physics update frame definition depends on the engine and game used. For Unity, it is the equivalent of "FixedUpdate". You should use Physic Update when you want to apply physics, e.g. when applying force to an object.

### ScheduleEveryAsyncFrame
 Schedules a task to run on every frame on a **seperate** thread. Running on async frames is vital for performance if your task is doing heavy calculations and causing delays. This will prevent the server from freezing in such cases.
```csharp
taskScheduler.ScheduleEveryAsyncFrame(this, SampleTask, "My custom Task");
```

> **Note**: Keep in mind that in most cases you can not access Game or Engine APIs from separate threads.

### ScheduleNextAsyncFrame
Schedules a task to be run on the next frame in a **seperate** thread once. Running on async frames is vital for performance if your task is doing heavy calculations and causing delays. This will prevent the server from freezing in such cases.

```csharp
taskScheduler.ScheduleNextAsyncFrame(this, SampleTask, "My custom Task");
```
> **Note**: Keep in mind that in most cases you can not access Game or Engine APIs from separate threads.

### ScheduleDelayed
Schedules a task to be run once after a set delay time. Can be configured to run the main thread or on a seperate thread.
```csharp
TimeSpan runAfter = TimeSpan.FromSeconds (15); // Run after 15 seconds
taskScheduler.ScheduleDelayed(this, SampleTask, "My custom Task", runAfter, true); // Async, runs on separate thread
taskScheduler.ScheduleDelayed(this, SampleTask, "My custom Task", runAfter); // Sync, runs on main thread
```

### ScheduleAt
Schedules a task to be run at the given time. The time is based on Utc timezone. Keep in mind that the execution time is not exact, but within a few milliseconds depending on the server load. 
```csharp
// Run after 15 seconds
DateTime runTime = DateTime.UtcNow + TimeSpan.FromSeconds (15);

taskScheduler.ScheduleAt(this, SampleTask, "My custom Task", runTime, true); // Async, runs on separate thread
taskScheduler.ScheduleAt(this, SampleTask, "My custom Task", runTime); // Sync, runs on main thread
```

### SchedulePeriodically
Schedules a task to be run every set amount of time and with an optional first delay. Can be configured to run on **main** thread or a **seperate** thread.
```csharp
TimeSpan runEvery = TimeSpan.FromSeconds(30); // Run every 30 seconds
TimeSpan runAfter = TuneSpan.FromSecond (5);  // After a first 5 seconds delay

taskScheduler.SchedulePeriodically(this, SampleTask, "My custom Task", runEvery, null, true); // Async, runs on a separate thread
taskScheduler.SchedulePeriodically(this, SampleTask, "My custom Task", runEvery, runAfter, true); // Delayed async
taskScheduler.SchedulePeriodically(this, SampleTask, "My custom Task", runEvery); // Sync, runs on main threasd
taskScheduler.SchedulePeriodically(this, SampleTask, "My custom Task", runEvery, runAfter); // Delayed sync
```

### RunOnMainThread
Schedules a task via `ScheduleNextFrame` if the current thread is no the main thread. Otherwise, it will run the task directly.

```csharp
taskScheduler.RunOnMainThread(this, SampleTask, "My custom Task");
```

## Extra
### Getting a List of Active Tasks

This will return all scheduled tasks.
```csharp
var pendingTasks = taskScheduler.Tasks;
```

### Cancelling a Task
```csharp
ITask myTask = taskScheduler.ScheduleEveryFrame(this, SampleTask, "My Task Description");
myTask.Cancel();
```

### Passing Arguments to Tasks when Scheduling
```csharp
protected override async Task OnActivate(bool isFromReload)
{
    string myArgument = "A little touch of wizard";

    taskScheduler.ScheduleNextFrame(this, () => SampleTask(myArgument), "Wizard's Sample Task");
}

public void SampleTask (string sampleParameter)
{
    Logger.LogInformation(sampleParameter);
}
```

### Returning Data with Callbacks
You can't not return directly from tasks, however you can create a callback.
```csharp
protected override async Task OnActivate(bool isFromReload)
{
    taskScheduler.ScheduleNextAsyncFrame (this, () => SampleTask(SampleTaskCallback), "SampleMethod");
}

public void SampleTask(Action<MyObject> callback)
{
    MyObject sampleData = // Do Something...

    // Run callback on same thread:
	callback?.Invoke(sampleData);

    // Run callback on main thread:
    if(callback != null)
    { 
        taskScheduler.RunOnMainThread(this, () => callback.Invoke(sampleData), "My Callback");
    }
}

public void SampleTaskCallback (MyObject sampleReturn)
{
    // Do some more things...
}
```

## Best Practices
Always use the TaskScheduler if you need to run tasks in a separate thread. The TaskScheduler provides RocketMod a uniform way to handle tasks in a consistent way. For example, this way RocketMod can dynamically schedule and run tasks based on current server load.
 
In order to achieve this,

* **Don't create your own threads.**

* **Don't use the Timer class for running tasks periodically.**

* **Do not use engine specific components like MonoBehaviour's for functions like Update(), FixedUpdate(), etc.**


Use the TaskScheduler instead.

Because RocketMod can not control custom plugin threads, this would result in lots of threads with no pooling. As a result, a lot of context switches would occur which would drastically slow down performance. Plugins also often do not create and handle threads correctly, resulting in endless loops and poor performance.

RocketMod uses `WaitHandle`s to ensure that threads do not run in empty loops and also dynamically schedules tasks to always ensure best performance.