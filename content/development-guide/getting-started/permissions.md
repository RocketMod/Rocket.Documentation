# Permissions
Permissions are used to determine which users can execute an action and which can not.

RocketMod exposes two permissions related services:

* IPermissionChecker: Used to check if a user has a requested permission

* IPermissionProvider: Provides permissions and manages them (e.g. adding groups, adding or removing permissions, etc...)

You can add these services like this to your plugin:
```csharp
namespace SamplePlugin
{
    public class MyPluginMain : Plugin
    {
        private IPermissionChecker permissionChecker;
        private IPermissionProvider permissionProvider;

        public MyPluginMain(IDependencyContainer container, IPermissionChecker permissionChecker, IPermissionProvider permissionProvider) : base ("MyPlugin", container)
        {
            this.permissionChecker = permissionChecker;
            this.permissionProvider = permissionProvider;
        }
    }
}
```
With the permission provider added we can begin going over its features.

## Managing Permissions
### Checking Permissions
To check if a user has a permission, use `CheckPermissionAsync`.

```csharp
PermissionResult checkResult = await permissionChecker.CheckPermissionAsync(user, "rocket");

switch(checkResult)
{
    case PermissionResult.Grant:
        // The permission was explicitly granted
        DoSomething();
        break;
    case PermissionResult.Deny:
        // The permission was explicitly denied
        ShowPermissionDeniedErrorMessage();
        break;
    case PermissionResult.Default:
        // The permission was not set up. Usually the action get's denied here as well.
        ShowPermissionDeniedErrorMessage();
        break;				
}
```

### Checking against All Permissions
To check if a user has access to all requested permissions, use `CheckHasAllPermissionsAsync`.
```csharp
if (await permissionChecker.CheckHasAllPermissionsAsync(user, "rocket", "p", "i") == PermissionResult.Grant)
{
    // Do Something
}	
```

### Checking against Any Permission
To check if a user has any permission from the requested permissions, use `CheckHasAnyPermissionAsync`.
```csharp
if (await permissionChecker.CheckHasAnyPermissionAsync(user, "rocket", "p", "i") == PermissionResult.Grant)
{
    // Do Something
}	
```

### Granting Permissions
To grant a permission to a group or user, use `AddPermissionAsync`.

```csharp
// will return true if successful
// Adds permission to user
await permissionProvider.AddPermissionAsync(user, "rocket");
// Adds permission to group
await permissionProvider.AddPermissionAsync(group, "rocket");
```

> This will result in `PermissionResult.Grant` return on permission check.

### Remove Permission
To remove a permission from a group or user, use `RemovePermissionAsync`.

```csharp
// will return true if removed else false
// Remove permission from user
await permissionProvider.RemovePermissionAsync(user, "rocket");
// Remove permission from group
await permissionProvider.RemovePermissionAsync(group, "rocket");
```

> This will result in `PermissionResult.Default` return on permission check.

### Add a explicitly Denied Permission
To deny a permission from a group or user use `AddDeniedPermissionAsync`. This will override any explicit grants, inherited or not.

```csharp
// returns true if added else false
// Add to user
await permissionProvider.AddDeniedPermissionAsync(user, "heal");
// Add to group
await permissionProvider.AddDeniedPermissionAsync(group, "heal");
```

> This will result in `PermissionResult.Denied` return on permission check.

### Removing a explicitly Denied Permission
To remove explicit denying of a permission, use `RemoveDeniedPermissionAsync`.

```csharp
// returns true if removed else false
// Remove from user
await permissionProvider.RemoveDeniedPermissionAsync(user, "heal");
// Remove from group
await permissionProvider.RemoveDeniedPermissionAsync(group, "heal");
```

> This will result in `PermissionResult.Default` return on permission check.

## Managing Permission Groups
### Getting a Group
To get a group by it's ID, use `GetGroupAsync`.

```csharp
IPermissionGroup group = await permissionProvider.GetGroupAsync("vip");
```

### Getting the Primary Permission Group of a User
To get a user's primary group, use `GetPrimaryGroupAsync`.

```csharp
IPermissionGroup group = await permissionProvider.GetPrimaryGroupAsync(user);
```

### Getting All Permission Groups
To get all permission groups, use `GetGroupsAsync`.

```csharp
IEnumerable<IPermissionGroup> groups = await permissionProvider.GetGroupsAsync();
```

### Getting Permission Groups of a User
To get permission groups of a user, use `GetGroupsAsync(user)`.
```csharp
IEnumerable<IPermissionGroup> groups = await permissionProvider.GetGroupsAsync(user);
```

### Add a User to a Permission Group
To add a user to a permission group, use `AddGroupAsync`.

```csharp
IPermissionGroup group = await permissionProvider.GetGroupAsync("vip");
if (group == null)
	return; // Group doesnt exist

await permissionProvider.AddGroupAsync(user, group);
```

### Removing a Permission Group
To remove a user from a permission group, use `RemoveGroupAsync`.

```csharp
IPermissionGroup group = permissionProvider.GetGroupAsync("vip");
if (group == null)
	return; // Group doesnt exist

await permissionProvider.RemoveGroupAsync(user, group);
```

### Creating a Permission Group
To create a new group, use `CreateGroupAsync`:

```csharp
await permissionProvider.CreateGroupAsync(new PermissionGroup() 
{
	Id = "vip",
	Name = "VIP",
	...
});
```

### Deleting a Permission Group
To delete a group, use `DeleteGroupAsync`:

```csharp
await permissionProvider.DeleteGroupAsync(group);
```

### Updating a Permission Group
To updates a group, use `UpdateGroupAsync`:

```csharp
var group = await permissionProvider.GetGroupAsync("vip");

// Note: It is not guaranteed that you can change a Group's ID after it has been set.
group.Name = "Gold";

await permissionProvider.UpdateGroupAsync(group);
```

### Loading, Saving and Reloading the Permissions

```csharp
await permissionProvider.LoadAsync(context); // load from a config context
await permissionProvider.SaveAsync(); // save to config
await permissionProvider.ReloadAsync(); // reload from config
```

## Best Practices
Usually you should deny actions when PermissionResult.Default is returned on requested permissions. Only do not deny if you want an action to be allowed by default, not denied.

Do not use permission checks in commands, use child commands instead, which always have their own permissions defined by RocketMod. 