# Black Division Spawn Mechanics - Complete Breakdown

This document provides an exact breakdown of how Black Division NPCs spawn in the game, including all trigger mechanisms, spawn chances, and configurations.

---

## Table of Contents
1. [Overview](#overview)
2. [Labs Spawn System](#labs-spawn-system)
3. [Hunt Trigger System](#hunt-trigger-system)
4. [Patrol System (Disabled)](#patrol-system-disabled)
5. [Checkpoint System (Disabled)](#checkpoint-system-disabled)
6. [Configuration System](#configuration-system)

---

## Overview

Black Division NPCs use **three spawn mechanisms**:

| Mechanism | Status | Maps | Trigger Type |
|-----------|--------|------|--------------|
| **Labs Spawns** | ✅ Active | Labs only | Static + EXFIL-triggered |
| **Hunt Mode** | ✅ Active | All other maps | Event-triggered (8% chance every 240s) |
| **Patrols** | ❌ Disabled | Would be configurable | Time + zone-based |
| **Checkpoints** | ❌ Disabled | Would be configurable | Static position-based |

---

## Labs Spawn System

### 1. Normal Labs Spawn

**Location**: `Server/Controllers/SpawnController.cs`, lines 60-81

```csharp
var normalSpawn = new BossLocationSpawn
{
    BossName = "blackDivAssault",
    BossChance = 15,                    // 15% chance per raid
    BossDifficulty = "normal",
    BossEscortAmount = "2,2,2,3,3,4",  // Weighted random: 50% for 2, 33% for 3, 17% for 4 escorts
    BossEscortType = "blackDivAssault",
    BossZone = "BotZoneFloor2,BotZoneFloor1,BotZoneBasement",
    Delay = 0,                          // Spawns immediately
    Time = -1,                          // Spawn time: -1 = raid start
    SpawnMode = ["regular", "pve"],
};
```

**How it works:**
- **Spawn Chance**: 15% chance to spawn at the start of every Labs raid
- **Team Size**: 1 leader + 2-4 escorts = **3-5 total NPCs**
- **Spawn Zones**: Random selection from Floor 2, Floor 1, or Basement
- **Timing**: Immediate spawn at raid start (Delay = 0, Time = -1)
- **Modes**: Works in both regular and PvE modes

### 2. Gate 1 EXFIL Spawn

**Location**: `Server/Controllers/SpawnController.cs`, lines 83-109

```csharp
if (randomUtil.GetChance100(20))  // 20% chance this gate is "armed"
{
    var exfilSpawn = new BossLocationSpawn
    {
        BossName = "blackDivAssault",
        BossChance = 100,                   // 100% spawn if armed
        BossEscortAmount = "3,3,4,5",       // Weighted random: 50% for 3, 25% for 4, 25% for 5 escorts
        BossZone = "BotZoneGate1",
        Delay = 8,                          // 8 second delay after trigger
        ForceSpawn = true,
        IgnoreMaxBots = true,
        TriggerId = "autoId_00632_EXFIL",   // Specific EXFIL point ID
        TriggerName = "interactObject"      // Triggers when player approaches/interacts
    };
}
```

**How it works:**
- **Activation**: 20% chance this gate is "armed" when the server starts
- **Spawn Chance**: If armed, 100% chance to spawn when triggered
- **Team Size**: 1 leader + 3-5 escorts = **4-6 total NPCs**
- **Trigger**: When player approaches/interacts with Gate 1 EXFIL (autoId_00632_EXFIL)
- **Timing**: 8 second delay after player triggers the EXFIL
- **Zone**: BotZoneGate1 (near the Gate 1 extraction point)
- **Special**: ForceSpawn = true, IgnoreMaxBots = true (always spawns, bypasses bot limits)

### 3. Gate 2 EXFIL Spawn

**Location**: `Server/Controllers/SpawnController.cs`, lines 111-137

```csharp
if (randomUtil.GetChance100(20))  // 20% chance this gate is "armed"
{
    var exfilSpawn = new BossLocationSpawn
    {
        BossName = "blackDivAssault",
        BossChance = 100,
        BossEscortAmount = "3,3,4,5",
        BossZone = "BotZoneGate2",
        Delay = 8,
        TriggerId = "autoId_00014_EXFIL",   // Different EXFIL point ID
        TriggerName = "interactObject"
    };
}
```

**How it works:**
- Identical to Gate 1, but for Gate 2 EXFIL point (autoId_00014_EXFIL)
- Independent 20% chance from Gate 1 (both, one, or neither can be armed)

**EXFIL Trigger Mechanism:**
- `TriggerName = "interactObject"` tells EFT to monitor player interactions
- `TriggerId = "autoId_00XXX_EXFIL"` links to specific extraction point objects in Labs
- When player enters EXFIL zone or interacts with the gate, EFT's spawn system activates
- 8 second delay gives time for dramatic entrance

---

## Hunt Trigger System

**This is how Black Division spawns on ALL OTHER MAPS** (Customs, Streets, Shoreline, Lighthouse, Factory, Interchange, Rezervbase, Labyrinth, Woods, etc.)

### Server-Side Configuration

**Location**: `Server/Controllers/SpawnController.cs`, lines 198-221

```csharp
private void AdjustHuntSpawnsForMap(string map, List<BossLocationSpawn> spawns)
{
    spawns.RemoveAll(x => x.TriggerId == "blackDivHunt");
    AddHuntToMap(map, spawns);
}

private void AddHuntToMap(string map, List<BossLocationSpawn> spawns)
{
    var patrolSize = randomUtil.GetInt(3, 4);  // 3 or 4 NPCs
    var patrol = GeneratePatrol(patrolSize, 100, false);
    
    patrol.Time = -1;
    patrol.BossZone = "";                       // No specific zone
    patrol.TriggerName = "botEvent";            // Event-triggered spawn
    patrol.TriggerId = "blackDivHunt";          // Hunt event identifier
    patrol.ForceSpawn = true;
    
    spawns.Add(patrol);
}
```

**Key Details:**
- **Team Size**: 3-4 NPCs (1 leader + 2-3 escorts)
- **Spawn Chance**: 100% *if the hunt event fires*
- **Zone**: Empty string = can spawn anywhere on the map
- **Trigger Type**: `"botEvent"` = event-driven, not time/location-based
- **Trigger ID**: `"blackDivHunt"` = the hunt event name

### Client-Side Hunt Manager

**Location**: `Plugin/Components/HuntManager.cs`

#### The Hunt Event Loop

```csharp
// Line 41: Every 240 seconds (4 minutes)
nextUpdate = Time.time + 240f;

// Line 43: 8% chance to start a hunt
if (UnityEngine.Random.Range(0, 100) < 8 && unpickedHunts.Count > 0)
{
    var randomEvent = unpickedHunts.Random();
    unpickedHunts.Remove(randomEvent);
    StartHunt(randomEvent);  // Triggers "blackDivHunt"
}
```

**How it works:**

1. **Timer**: Every 240 seconds (4 minutes), HuntManager checks if it should start a hunt
2. **Chance**: 8% chance to trigger a hunt event
3. **Event Pool**: Uses `unpickedHunts` list (initialized with "blackDivHunt" at raid start)
4. **One-Time**: Each hunt event can only fire once per raid (removed from unpickedHunts)

#### What Actually Triggers the Hunt

```csharp
// Line 52-57
public void StartHunt(string huntEvent)
{
    huntRole = huntEvents[huntEvent];              // huntEvents["blackDivHunt"] = (WildSpawnType)848421
    startNewHunt = true;
    Singleton<BotEventHandler>.Instance.AnyEvent(huntEvent);  // ← THIS TRIGGERS THE SPAWN
    Plugin.LogSource.LogInfo($"[BlackDiv] Starting hunt event {huntEvent}");
}
```

**Critical Line**: `BotEventHandler.Instance.AnyEvent("blackDivHunt")`

- **BotEventHandler** is an EFT engine class that broadcasts bot events
- `AnyEvent()` notifies the bot spawning system that a special event occurred
- EFT's spawn controller finds all spawn configs with `TriggerId = "blackDivHunt"`
- Those spawn configs are activated, creating the bot groups

#### After Bots Spawn

```csharp
// Line 72-92: When any bot spawns
public void OnBotCreated(BotOwner bot)
{
    if (!WildSpawnTypeExtensions.IsBlackDiv(bot.Profile.Info.Settings.Role)) return;
    
    var huntManager = bot.gameObject.GetOrAddComponent<BotHuntManager>();
    huntManager.Init(bot, this);
    FindFirstHuntTarget(huntManager);  // Assigns a target to hunt
}
```

**Hunt Target Selection** (lines 99-131):
- Searches all alive players in raid
- Valid targets for blackDivAssault (848421):
  - PMCs: USEC or BEAR players (line 192)
  - Bots: pmcUSEC or pmcBEAR bot types (line 196)
- All bots in the group hunt the same target
- If target dies, finds a new target (line 133-170)

### Hunt Behavior

**Location**: `Plugin/Behavior/HuntTargetAction.cs` and `Plugin/Behavior/HuntTargetLayer.cs`

Once a Black Division bot has a hunt target:
1. **HuntTargetLayer** activates (behavior layer system)
2. **HuntTargetAction** executes the hunting behavior
3. Bot navigates toward the target's last known position
4. Uses regroup logic to keep the squad together
5. Engages target when in range

---

## Patrol System (Disabled)

**Location**: `Server/Controllers/SpawnController.cs`, lines 164-196

The patrol system exists in the code but is **never called** because:
1. Configuration loading is disabled (line 53 in `Mod.cs`): `//ModConfig = _modHelper.GetJsonDataFromFile<MainConfig>(...)`
2. `AdjustPatrolSpawnsForMap()` method exists but is never invoked

### How Patrols Would Work (If Enabled)

```csharp
private void AdjustPatrolSpawnsForMap(string map, MapConfig mapConfig, MainConfig mainConfig, List<BossLocationSpawn> spawns)
{
    var patrolConfig = mapConfig.patrol;
    
    for (int i = 0; i < patrolConfig.patrolAmount; i++)
    {
        var patrolSize = randomUtil.GetInt(patrolConfig.patrolMin, patrolConfig.patrolMax);
        var patrol = GeneratePatrol(patrolSize, patrolConfig.patrolChance);
        
        patrol.BossZone = randomUtil.GetArrayValue(validZones);  // Random zone
        patrol.Time = randomUtil.GetInt(patrolConfig.patrolTimeMin, patrolConfig.patrolTimeMax);
        
        spawns.Add(patrol);
    }
}
```

**Configuration Schema** (from `Server/Models/MainConfig.cs`, lines 26-36):

```csharp
public class MapPatrolConfig
{
    public bool enablePatrols { get; set; }
    public float patrolChance { get; set; }        // Spawn chance per patrol (0-100 percentage)
    public int patrolAmount { get; set; }          // Number of patrol groups
    public int patrolMin { get; set; }             // Min patrol size
    public int patrolMax { get; set; }             // Max patrol size
    public List<string> patrolZones { get; set; }  // Valid spawn zones
    public int patrolTimeMin { get; set; }         // Min spawn time (seconds)
    public int patrolTimeMax { get; set; }         // Max spawn time (seconds)
}
```

**Example config.jsonc structure** (would need to be created):

```jsonc
{
  "debug": {
    "logs": false,
    "spawnAlways": false,           // Forces 100% spawn chance if true
    "spawnInstantlyAlways": false   // Forces immediate spawns if true
  },
  "locations": {
    "customs": {
      "patrol": {
        "enablePatrols": true,
        "patrolChance": 30,          // 30% chance per patrol group
        "patrolAmount": 2,           // 2 patrol groups
        "patrolMin": 3,              // 3-5 NPCs per patrol
        "patrolMax": 5,
        "patrolZones": ["ZoneScavBase", "ZoneDorms", "ZoneGasStation"],
        "patrolTimeMin": 60,         // Spawn 60-300 seconds into raid
        "patrolTimeMax": 300
      }
    }
  }
}
```

---

## Checkpoint System (Disabled)

**Location**: Config schema exists in `Server/Models/MainConfig.cs`, lines 38-55

```csharp
public class MapCheckpointConfig
{
    public bool enableCheckpoints { get; set; }
    public int checkpointAmount { get; set; }
    public List<ZoneCheckpointConfig> checkpointZones { get; set; }
}

public class ZoneCheckpointConfig
{
    public string checkpointZone { get; set; }
    public float checkpointChance { get; set; }    // Spawn chance (0-100 percentage)
    public int checkpointMin { get; set; }
    public int checkpointMax { get; set; }
    public float checkpointRadius { get; set; }
    public float x { get; set; }  // 3D coordinates
    public float y { get; set; }
    public float z { get; set; }
}
```

**Purpose**: Checkpoints would create static defensive positions at specific coordinates with a radius, but **no code implements this system** - only the config schema exists.

---

## Configuration System

### Current State

**Line 53 in `Server/Mod.cs`:**
```csharp
//ModConfig = _modHelper.GetJsonDataFromFile<MainConfig>(pathToMod, "config.jsonc");
```

**This line is commented out**, meaning:
- No `config.jsonc` file is loaded
- Patrol system cannot be configured
- Checkpoint system cannot be configured
- Only hard-coded spawns (Labs + Hunt) are active

### To Enable Configuration

1. Create `config.jsonc` in the mod's root directory (next to `Server/` folder)
2. Uncomment line 53 in `Server/Mod.cs`
3. Call `AdjustPatrolSpawnsForMap()` in the spawn controller
4. Populate config with patrol/checkpoint settings per map

---

## Summary: Why You Rarely See Black Division NPCs

### Labs Map
- **15% base spawn chance** is relatively low
- **20% chance per EXFIL gate** means only 1 in 5 raids have armed exits
- **Combined probability**: Chance of encountering at least one group = 1 - P(none spawn) = 1 - (0.85 × 0.8 × 0.8) ≈ **45.6% per Labs raid**

### Other Maps (Hunt Mode)
- **8% chance every 4 minutes** = very infrequent checks
- **One hunt per raid maximum** (removed from unpickedHunts after firing)
- **Early raid**: If the first check at 4 minutes fails (92% chance), no hunt
- **Late raid**: Multiple checks increase odds, but still low overall
- **Estimated probability**: ~8-20% depending on raid length

### Disabled Systems
- **Patrols**: Would provide consistent, time-based spawns with configurable chances
- **Checkpoints**: Would provide static defensive positions
- Both disabled due to commented config loading

---

## Spawn Probability Calculations

### Labs (per raid)
- Normal spawn: **15%**
- At least one armed EXFIL: **1 - (0.8 × 0.8) = 36%**
- Both EXFILs armed: **0.2 × 0.2 = 4%**
- **Encounter at least one group**: ~**42-45%** (accounting for overlap)

### Hunt Mode (15 minute raid)
- Checks at: 4min, 8min, 12min
- Probability of at least one success: **1 - (0.92³) ≈ 22%**

### Hunt Mode (25 minute raid)
- Checks at: 4min, 8min, 12min, 16min, 20min, 24min
- Probability: **1 - (0.92⁶) ≈ 39%**

---

## Technical Details

### Bot Type IDs
```csharp
848420 = blackDivLead      // Leader role
848421 = blackDivAssault   // Assault role (used for most spawns)
848422 = blackDivBreacher  // Breacher role
848423 = blackDivSupport   // Support role
```

### Spawn Modes
```csharp
SpawnMode = ["regular", "pve"]
```
Works in both regular SPT and PvE mode.

### Force Spawn vs Normal Spawn
- **ForceSpawn = true**: Always spawns, ignores bot limits (used for EXFIL and Hunt)
- **ForceSpawn = false**: Respects map's max bot count (used for Labs normal spawn)

### IgnoreMaxBots
- **IgnoreMaxBots = true**: Can spawn even if map is at bot capacity
- **IgnoreMaxBots = false**: Will not spawn if too many bots exist

---

## Files Reference

| File | Purpose |
|------|---------|
| `Server/Controllers/SpawnController.cs` | Server spawn configuration |
| `Server/Models/MainConfig.cs` | Configuration schema |
| `Server/Mod.cs` | Mod initialization, config loading |
| `Plugin/Components/HuntManager.cs` | Hunt event system, target assignment |
| `Plugin/Components/BotHuntManager.cs` | Per-bot hunt state tracking |
| `Plugin/Behavior/HuntTargetLayer.cs` | Hunt behavior activation |
| `Plugin/Behavior/HuntTargetAction.cs` | Hunt movement/engagement logic |

---

*This documentation is based on source code analysis of the Black Division mod.*
