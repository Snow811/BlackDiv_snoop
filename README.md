# BlackDiv
Adds Black Division using MoreBotsAPI

## Spawn Mechanics

### How Black Division NPCs Spawn

The Black Division mod currently spawns NPCs primarily on the **Labs** map with three different spawn types:

#### 1. Normal Labs Spawn
- **Spawn Chance**: 45% per raid
- **Location**: Floors 1, 2, and Basement zones
- **Team Size**: 3-5 NPCs (1 leader + 2-4 escorts)
- **Timing**: Immediate (spawns at raid start)
- **Bot Type**: blackDivAssault

#### 2. Gate 1 EXFIL Spawn
- **Activation Chance**: 50% (rolled when server starts)
- **Spawn Chance**: 100% (if activated)
- **Location**: Gate 1 exit area (BotZoneGate2)
- **Team Size**: 4-6 NPCs (1 leader + 3-5 escorts)
- **Timing**: 8 seconds after approaching the exit
- **Trigger**: Interacting with Gate 1 EXFIL
- **Bot Type**: blackDivAssault

#### 3. Gate 2 EXFIL Spawn
- **Activation Chance**: 50% (rolled when server starts)
- **Spawn Chance**: 100% (if activated)
- **Location**: Gate 2 exit area (BotZoneGate1)
- **Team Size**: 4-6 NPCs (1 leader + 3-5 escorts)
- **Timing**: 8 seconds after approaching the exit
- **Trigger**: Interacting with Gate 2 EXFIL
- **Bot Type**: blackDivAssault

#### 4. Hunt Mode (Other Maps)
On other maps (Customs, Streets, Shoreline, Lighthouse, Factory, Interchange, etc.), Black Division operates in "Hunt" mode:
- **Spawn Method**: Triggered by bot events (not random spawns)
- **Team Size**: 3-4 NPCs
- **Trigger ID**: "blackDivHunt"
- This requires specific in-game events to spawn, which is why you rarely see them outside Labs

### Why You Rarely See Black Division NPCs

1. **Limited to Labs**: Regular spawns only occur on Labs map
2. **Previous Low Spawn Rate**: The normal spawn was only 15% (now increased to 45%)
3. **EXFIL Spawns Are Random**: Only a 50% chance the exits will be "armed" with BlackDiv
4. **Other Maps Need Events**: Hunt mode on other maps requires specific triggers

### Adjusting Spawn Rates

To modify spawn chances, edit `/Server/Controllers/SpawnController.cs`:

- **Line 64**: `BossChance = 45` - Normal Labs spawn percentage
- **Line 83**: `randomUtil.GetChance100(50)` - Gate 1 EXFIL activation chance
- **Line 111**: `randomUtil.GetChance100(50)` - Gate 2 EXFIL activation chance

Higher numbers = more frequent spawns (0-100 scale)
