## Administration Commands

### /rocket

**Supported Caller:** All

**Summary:** Manages RocketMod.

**Syntax:** \<reload\>

**Permission:** Rocket.ManageRocket
  
---

### /permission (/p)

**Supported Caller:** All

**Summary:** Manages Rocket permissions.

**Syntax:** [add | remove | reload]

**Permission:** Rocket.Permissions.ManagePermissions

---

### /permissionGroup (/pg)

**Supported Caller:** All

**Summary:** Manages permission groups.

**Syntax:** [add | remove]

**Permission:** Rocket.Permissions.ManageGroups

---

## Tool Commands

### /help

**Supported Caller:** All

**Summary:** Provides help for all or a specific command.

**Syntax:** [command] [1. Child Command] [2. Child Command] [...]

**Permission:** Rocket.Help

---

## Migration Commands

### /migrateconfig

**Supported Caller:** Console

**Summary:** Migrates configs from one type to another.

**Syntax:** [<from type> <to type> <path>]

**Permission:** Rocket.Migrate.Config

---

### /legacyMigration

**Supported Caller:** Console

**Summary:** Migrates from old RocketMod 4.

**Syntax:** [step]

**Permission:** Rocket.Migrate.Legacy

---
