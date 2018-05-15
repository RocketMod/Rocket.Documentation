## Administration Commands

### /save

**Supported Caller:** All

**Summary:** Saves all vanilla server data.

**Permission:** Rocket.Save

---

### /shutdown (/stop, /quit)

**Supported Caller:** All

**Summary:** Saves all vanilla server data, kicks all users, then stops the server.

**Permission:** Rocket.Shutdown

---

### /kick

**Supported Caller:** All

**Summary:** Kicks a player.

**Syntax:** [[n]ame / id] [reason]

**Note:** Target must be online.

**Permission:** Rocket.Kick

---

### /ban

**Supported Caller:** All

**Summary:** Bans a user.

**Syntax:** [[n]ame / id] [reason]
  
**Note:** Target must be online to use their name.

**Permission:** Rocket.Ban

---

### /admin

**Supported Caller:** All

**Summary:** Gives a player admininstrator priveleges.

**Syntax:** [[name] / id]

**Note:** Target must be online.

**Permission:** Rocket.Admin

---

### /unadmin (deladmin, removeadmin, deadmin)

**Supported Caller:** All

**Summary:** Strips a player of their administrator priveleges.

**Syntax:** [[n]ame / id]

**Note:** Target must be online to use their name.

**Permission:** Rocket.RemoveAdmin

---

## Cheat Commands

### /feed (/eat)

**Supported Caller:** All

**Summary:** Makes you or another player eat.

**Syntax:** [[n]ame / id]

**Note:** Must target user if you're using the Console.

**Permission:** Rocket.Feed

---

### /skills (experience, exp)

**Supported Caller:** All

**Summary:** Give yourself or someone else skill points.

**Syntax:** [[n]ame / id] [points] | [points]

**Note:** Must target user if you're using the Console.

**Permission:** Rocket.Skills

---

## Vanilla or Modkit Commands

**Supported Caller:** Must be called from an in-game user.

**Summary:** *Command-Specified*

**Syntax:** [arg1],[arg2],[...]

**Permission:** Eco.Base.[Command]
