> **Important: The examples in here may be outdated, feel free to help us keeping them up to date, you can simply edit them if you are logged in on GitHub.**

***
## Base
We need to first add the permission provider

```csharp
namespace SamplePlugin
{
	public class Main : Plugin
	{
		private IPermissionProvider permissionProvider;
		protected Main (IDependencyContainer container, IPermissionProvider) : base ("SamplePlugin", container)
		{
			this.permissionProvider = permissionProvider;
		}
	}
}
```
With the permission provider added we can begin going over its features.

## Permissions
### Check Permission
Check if a user has a permission

```csharp
// Checks if they have the permission
if (permissionProvider.CheckPermission (user, "rocket") == PermissionResult.Grant)
	// Do Something

// Checks if they dont have the permission
if (permissionProvider.CheckPermission (user, "rocket") == PermissionResult.Deny)
	//Do Sometin
```

### Check All Permissions
Check if a user has an array of permissions

```csharp
if (permissionProvider.CheckHasAllPermissions (user, "rocket", "p", "i") == PermissionResult.Grant)
	// Do Something

if (permissionProvider.CheckHasAllPermissions (user, new string [] { "rocket", "p", "i" }) == PermissionResult.Grant)
	// Do Something
```

### Check Any Permission
Check if a user has any permission from an array

```csharp
if (permissionProvider.CheckHasAnyPermission (user, "rocket", "p", "i") == PermissionResult.Grant)
	// Do Something

if (permissionProvider.CheckHasAnyPermission (user, new string [] { "rocket", "p", "i" }) == PermissionResult.Grant)
	// Do Something
```

### Add Permission
Add a permission to a group or user

```csharp
// will return true if added else false
// Adds permission to user
permissionProvider.AddPermission (user, "rocket");
// Adds permission to group
permissionProvider.AddPermission (group, "rocket");
```

### Remove Permission
Remove a permission from a group or user

```csharp
// will return true if removed else false
// Remove permission from user
permissionProvider.RemovePermission (user, "rocket");
// Remove permission from group
permissionProvider.RemovePermission (group, "rocket");
```

### Add Denied Permission
Deny a permission from a group or user (full override only way to restore permission is to remove the deny)

```csharp
// returns true if added else false
// Add to user
permissionProvider.AddDeniedPermission (user, "heal");
// Add to group
permissionProvider.AddDeniedPermission (group, "heal");
```

### Remove Denied Permission
Remove the deny of a permission

```csharp
// returns true if removed else false
// Remove from user
permissionProvider.RemoveDeniedPermission (user, "heal");
// Remove from group
permissionProvider.RemoveDeniedPermission (group, "heal");
```

## Groups
### Get Group
Get a group by Id

```csharp
IPermissionGroup group = permissionProvider.GetGroup ("vip");
```

### Get Primary Group
Get a users primary group

```csharp
IPermissionGroup group = permissionProvider.GetPrimaryGroup (user);
```

### Get Groups
Get all groups

```csharp
// Returns all groups
IEnumerable<IPermissionGroup> groups = permissionProvider.GetGroups ();

// Returns all groups a user has
IEnumerable<IPermissionGroup> groups = permissionProvider.GetGroups (user);
```

### Add Group
Add a group to a user

```csharp
IPermissionGroup group = permissionProvider.GetGroup ("vip");
if (group == null)
	return; // Group doesnt exist
permissionProvider.AddGroup (user, group);
```

### Remove Group
Remove a group from a user

```csharp
IPermissionGroup group = permissionProvider.GetGroup ("vip");
if (group == null)
	return; // Group doesnt exist
permissionProvider.RemoveGroup (user, group);
```

### Create Group
Create a new group

```csharp
permissionProvider.CreateGroup (new PermissionGroup () 
{
	Id = "vip",
	Name = "VIP",
	etc
});
```

### Delete Group
Delete a group

```csharp
permissionProvider.DeleteGroup (group);
```

### Update Group
Update a group

```csharp
var group = permissionProvider.GetGroup ("vip");
group.Name = "Gold";

permissionProvider.UpdateGroup (group);
```

### Load, Save, Reload

```csharp
permissionProvider.Load (context);
permissionProvider.Save ();
permissionProvider.Reload ();
```

## Creating your own Permission Provider
### Make new Class
Create the Permission Provider and implement its features. You can see examples from the [ConfigurationPermissionProvider] (https://github.com/RocketMod/Rocket/blob/master/Rocket.Core/Permissions/ConfigurationPermissionProvider.cs) and [ConsolePermissionProvider](https://github.com/RocketMod/Rocket/blob/master/Rocket.Core/Permissions/ConsolePermissionProvider.cs)

```csharp
namespace SamplePlugin
{
	[ServicePriority (Priority = ServicePriority.High)]
	public class MySamplePermissionProvider : IPermissionProvider
	{
		public MySamplePermissionProvider ()
		{

		}
		
		public string ServiceName => "SampleProvider";

		public bool SupportsTarget (IIdentity target)
		{
			// Implement	
		}

		public PermissionResult CheckPermission (IIdentity target, string permission)
		{
			// Implement	
		}

		public PermissionResult CheckHasAllPermissions (IIdentity target, params string [] permissions)
		{
			// Implement	
		}

		public PermissionResult CheckHasAnyPermission (IIdentity target, params string [] permissions)
		{
			// Implement	
		}

		public bool AddDeniedPermission (IIdentity target, string permission)
		{
			// Implement	
		}

		public bool AddGroup (IIdentity target, IPermissionGroup group)
		{
			// Implement	
		}

		public bool AddPermission (IIdentity target, string permission)
		{
			// Implement	
		}

		public bool CreateGroup (IPermissionGroup group)
		{
			// Implement	
		}

		public bool DeleteGroup (IPermissionGroup group)
		{
			// Implement	
		}

		public IPermissionGroup GetGroup (string id)
		{
			// Implement	
		}

		public IEnumerable<IPermissionGroup> GetGroups (IIdentity target)
		{
			// Implement	
		}

		public IEnumerable<IPermissionGroup> GetGroups ()
		{
			// Implement	
		}

		public IPermissionGroup GetPrimaryGroup (IUser user)
		{
			// Implement	
		}

		public void Load (IConfigurationContext context)
		{
			// Implement	
		}

		public void Reload ()
		{
			// Implement	
		}

		public bool RemoveDeniedPermission (IIdentity target, string permission)
		{
			// Implement	
		}

		public bool RemoveGroup (IIdentity target, IPermissionGroup group)
		{
			// Implement	
		}

		public bool RemovePermission (IIdentity target, string permission)
		{
			// Implement	
		}

		public void Save ()
		{
			// Implement	
		}

		public bool UpdateGroup (IPermissionGroup group)
		{
			// Implement	
		}
	}
}
```

### Register your provider

```csharp
namespace SamplePlugin
{
	public class Main : Plugin
	{
		protected Main (IDependencyContainer container) : base ("Sample Plugin", container)
		{
			container.RegisterSingletonType <IPermissionProvider, MySamplePermissionProvider> ();
		}
	}
}
```
