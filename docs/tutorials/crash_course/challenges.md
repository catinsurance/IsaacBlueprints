---
article: Challenges
authors: benevolusgoat
blurb: Learn how to create custom challenges.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - REPENTOGON
    - XML
    - Lua
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Challenges change the gameplay of a regular run for a unique twist. This tutorial will cover how to create your own, and all of the attributes you can modify that are unique to challenges.

## Creating your challenge

Creating a challenge requires a [challenges.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/challenges.html) file in your `content` folder. They require the root `challenges` tag and its child `challenge` tag, where you define your individual challenges. Below are explanations for their variables:

???- info "Root `challenges` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| version | int | The version of the `challenges.xml` format. **This must always stay equal to 1**. |

???- info "`challenge` tag variables"
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	|id|int|In-game id of the challenge (Used only by vanilla challenges. Use the `name` variable instead.)|
	|name|string|Name of the challenge.|
	|startingitems|string|A comma separated list of ids for items granted to the player at the start.|
	|startingitems2|string|A comma separated list of ids for items granted to player 2 (Esau when playing as Jacob & Esau) at the start.|
	|startingtrinkets|string|Comma separated list of ids for trinkets that will be granted to the player at the start (max 2).|
	|startingcard|string| [Card id](https://wofsauge.github.io/IsaacDocs/rep/enums/Card.html) of the starting cards.<br>Default is `-1` (no card).|
	|startingpill|string| [PillEffect id](https://wofsauge.github.io/IsaacDocs/rep/enums/PillEffect.html) of the starting pill.<br>Default is `-1` (no pill).|
	|playertype|string|[Player type id](https://wofsauge.github.io/IsaacDocs/rep/enums/PlayerType.html).<br>Default is `0` (Isaac).|
	|endstage|string|The last stage of the challenge (uses [LevelStage](https://wofsauge.github.io/IsaacDocs/rep/enums/LevelStage.html) internal ids).|
	|roomfilter|string|Comma separated list of [RoomTypes](https://wofsauge.github.io/IsaacDocs/rep/enums/RoomType.html) to blacklist from generating (not all room types can be used).|
	|cursefilter|int|Bitmask of [curses](https://wofsauge.github.io/IsaacDocs/rep/enums/LevelCurse.html) to blacklist from appearing.<br>(Darkness = 1, Labyrinth = 2, Lost = 4, Unknown = 8, Cursed = 16, Maze = 32, Blind = 64, Giant = 128)|
	|getcurse|int|Bitmask of curses to always appear (same ids as `cursefilter`).|
	|achievements|string|List of achievement ids that are required to be able to play the challenge.|
	|altpath|bool|Determines if the player takes the light or dark path. Omit to allow either path, `true` to force Cathedral/Chest path, or `false` to force Sheol/Dark Room path.|
	|canshoot|bool|Determines if player can shoot.<br>Default is `true` (shooting enabled). Cannot force characters who have `canshoot` set to `false` in their `players.xml` file to shoot, such as Lilith.|
	|redhp|int|Adds red hearts to the base of the chosen character. 1 = half heart. Negative numbers can be used to remove hp.|
	|maxhp|int|Adds red heart containers to the base of the chosen character. 2 = One red heart container. Negative numbers can be used to remove containers.|
	|soulhp|int|Adds soul hearts to the base of the chosen character. 1 = half soul heart.|
	|blackhp|int|Adds black hearts to the base of the chosen character. 1 = half black heart.|
	|coins|int|Adds starting coins.|
	|maxdamage|bool|If set to `true`, damage cannot go above `100`.|
	|adddamage|float|Adds damage to the player.|
	|minfirerate|float|Sets starting fire delay (not the same as the in-game tears stat, [see here](../basics/tears_up.md)).|
	|minshotspeed|bool|Sets the maximum shot speed to `1.00`.|
	|bigrange|bool|Grants an additional 10 range.|
	|difficulty|int|[Game difficulty](https://wofsauge.github.io/IsaacDocs/rep/enums/Difficulty.html) (0: normal (default), 1: hard, 2: Greed, 3: Greedier)<br>Greed and greedier mode work, but when killing Ultra Greed, the big Chest spawns, instead of a trophy.|
	|megasatan|bool|Last boss is Mega Satan. Adds both key pieces to the player.|
	|secretpath|bool|Force the Repentance alt path.|

:modding-repentogon: REPENTOGON modifies some of the existing variables to allow for using modded content and adds its own set of `challenge` tag variables.

???- info ":modding-repentogon: REPENTOGON-exclusive `challenge` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	|unlockachievement|string|Sets the achievement (preferably a modded one using the name of the achievement) to unlock when the challenge is completed.|
	|startingitems|string|This now supports modded items by using the names of the items instead of the ids. Same format as in vanilla, a comma separated list of values.|
	|startingitems2|string|This now supports modded items by using the names of the items instead of the ids. Same format as in vanilla, a comma separated list of values.|
	|startingtrinkets|string|This now supports modded trinkets by using the names of the trinkets instead of the ids. Same format as in vanilla, a comma separated list of values.|
	|playertype|string|This now supports modded characters by using the names of the items instead of the ids.|
	|hidden|boolean|Anything but "false" would result in a hidden challenge that wont show up in the menu.|
	|achievements|string|Same as vanilla, comma separated list of achievements required to unlock the challenge. Use achievement names for modded achievements.|
	|lockeddesc|string|Hint to display instead of the challenge name when its locked. The default phrase is "LOCKED :(".|

### Example "challenges.xml" file:

This creates a new challenge called "My challenge" in the custom challenges tab, which ends after Mom's heart/It Lives. The player starts with Breakfast, Dead Cat and Little Steven, but cant shoot. Treasure rooms and Curse of Darkness are disabled.
```XML
<challenges version="1">
    <challenge playertype="0" name="My challenge" endstage="8" startingitems="25,81,100" roomfilter="1" cursefilter="1" canshoot="false" />
</challenges>
```

## Making modifications through Lua
Many variables available in the `challenges.xml` allow you to change attributes about challenges, however this is limited in what it can provide, and does not support modded content. This will require Lua code to change.

The first step is to get the ID of your challenge with [Isaac.GetChallengeIdByName](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getchallengeidbyname) and checking it against [Isaac.GetChallenge](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getchallenge). You can make any desired changes to your challenge through a simple `if` check to see if your challenge is active.

```Lua
local mod = RegisterMod("My Challenge Mod", 1)

--Fetch your challenge's ID
local MY_CHALLENGE = Isaac.GetChallengeIdByName("My new challenge")

function mod:OnGameStart(wasContinued)
	--Fetch the current challenge ID
	local challenge = Isaac.GetChallenge()
	--Checks that the challenge is our challenge, and is not from a continued run.
	if challenge == MY_CHALLENGE and not wasContinued then
		--Whatever you'd like here.
	end
end

--Will run when starting or continuing a run
mod:AddCallback(ModCallbacks.MC_POST_GAME_STARTED, mod.OnGameStart)
```

## First-time modifications to the player
You may want to include modded content in your challenge on the player when they first spawn in your challenge. You will need to account for multiple players, doing it only when starting the challenge as opposed to continuing it, and potentially running this code again when using the Genesis collectible.

### Non-REPENTOGON
Without REPENTOGON, special checks will need to be implemeneted. The following criteria must be met so that this will only trigger **once** per player when appropriate.

1. The room has only been visited once. Checked with [Room:IsFirstVisit](https://wofsauge.github.io/IsaacDocs/rep/Room.html#isfirstvisit).
2. The current floor is the very first floor. Can check [Level:GetStage](https://wofsauge.github.io/IsaacDocs/rep/Level.html#getstage) against [LevelStage](https://wofsauge.github.io/IsaacDocs/rep/enums/LevelStage.html).
3. The player is inside the starting room. Can check  [Level:GetCurrentRoomIndex](https://wofsauge.github.io/IsaacDocs/rep/Level.html#getcurrentroomindex) against [Level:GetStartingRoomIndex](https://wofsauge.github.io/IsaacDocs/rep/Level.html#getstartingroomindex).

It is also important to note that some things may need to be reapplied when using Genesis, such as re-adding items that were removed from the player. This can be a separate check to inact different behavior.

This snippet of code will transform the current character into a new custom character "[Gabriel](character.md)" with a new item "[Damage Potion](passive_item.md)".
```Lua
local mod = RegisterMod("My Challenge Mod", 1)

local game = Game()
local MY_CHALLENGE = Isaac.GetChallengeIdByName("My new challenge")
local PLAYER_GABRIEL = Isaac.GetPlayerTypeByName("Gabriel", false)
local DAMAGE_POTION = Isaac.GetItemIdByName("Damage Potion")

--Function passes the player as its first and only argument
function mod:OnPlayerInit(player)
	local challenge = Isaac.GetChallenge()
	local level = game:GetLevel()
	local room = game:GetRoom()
	local stage = level:GetStage()
	local curIndex = level:GetCurrentRoomIndex()
	local inGenesisRoom = curIndex == GridRooms.ROOM_GENESIS_IDX

	if challenge == MY_CHALLENGE
		--First visit to the room
		and room:IsFirstVisit()
		and (
			--Is the player in Floor 1, and they're in the starting room?
			(level:GetStage() == LevelStage.STAGE1_1 and curIndex == level:GetStartingRoomIndex())
			--Or did the player enter the Genesis room?
			or inGenesisRoom
		)
	then
		--Genesis' default behavior does not enforce the starting character
		if not isGenesisRoom then
			player:ChangePlayerType(PLAYER_GABRIEL)
		end
		player:AddCollectible(DAMAGE_POTION)
	end
end

--Runs when the player initializes from run start, run continue, or Genesis
mod:AddCallback(ModCallbacks.MC_POST_PLAYER_INIT, mod.OnPlayerInit)
```

### :modding-repentogon: REPENTOGON
All of the aforementioned implementations of modded content can be done directly through the `challenges.xml` with REPENTOGON. If there's anything that can't be done through the XML, no special checks are necessary, as the [MC_PLAYER_INIT_POST_LEVEL_INIT_STATS](https://repentogon.com/enums/ModCallbacks.html#mc_player_init_post_level_init_stats) callback does all that for you.

This `challenges.xml` takes the [previous XML](challenges.md#example-challengesxml-file) and modifies the variables to start the challenge as "[Gabriel](character.md)" with a new item "[Damage Potion](passive_item.md)".
```XML
<challenges version="1">
    <challenge playertype="Gabriel" name="My new challenge" endstage="8" startingitems="Damage Potion" roomfilter="1" cursefilter="1" canshoot="false" />
</challenges>
```

This Lua code will spawn a red heart for each player upon starting the challenge.
```Lua
local mod = RegisterMod("My Challenge Mod", 1)

local game = Game()
local MY_CHALLENGE = Isaac.GetChallengeIdByName("My new challenge")

--Function passes the player as its first and only argument
function mod:OnPlayerInit(player)
	local challenge = Isaac.GetChallenge()

	if challenge == MY_CHALLENGE then
		local room = game:GetRoom()
		local pos = room:FindFreePickupSpawnPosition(player.Position, 40)
		Isaac.Spawn(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_HEART, HeartSubType.HEART_FULL, pos, Vector.Zero, player)
	end
end

--Runs when the player initializes from run start, run continue, or Genesis
mod:AddCallback(ModCallbacks.MC_PLAYER_INIT_POST_LEVEL_INIT_STATS, mod.OnPlayerInit)
```