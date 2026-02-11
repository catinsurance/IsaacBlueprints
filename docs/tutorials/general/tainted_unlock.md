---
article: Tainted unlock
authors: benevolusgoat
blurb: Learn how to create the traditional unlock method for tainted characters.
comments: true
tags:
    - Tutorial
    - Intermediate
    - Repentance
    - Repentance+
    - REPENTOGON
    - Lua
---

{% include-markdown "hidden/unfinished_notice.md" start="<!-- start -->" end="<!-- end -->" %}

## Introduction

Tainted characters are alternate, twisted versions of their regular variants that are unlocked by finding them inside the Home floor's secret closet while playing as said regular variant. This article will cover setting up this unlock method for your own custom tainted character.

![Home Closet](../assets/tainted_unlock/closet.png)

## Requirements

For this tutorial, you will need:

1. [A regular custom character](../crash_course/character.md).
2. [A tainted variant of the character](../crash_course/character.md#creating-a-tainted-character).
3. Save data to to remember the state of the unlock.

A `boolean` value should be saved to remember the state of the unlock, for whether the tainted character is unlocked or not. :modding-repentogon: With REPENTOGON, you can create an [achievement](../repentogon/achievements.md) and attach it to your tainted character directily in the [players.xml](https://repentogon.com/xml/players.html) file. Without REPENTOGON, you will need to learn how to manually handle [save data](../concepts/saving_data.md).

## Locking access to the character

Before the tainted character can be unlocked, it must be locked and unable to be played. :modding-repentogon: As mentioned previously, REPENTOGON makes this process simple by allowing you to attach an achievement to the character, which will stop your character from being selected on the character selection menu. Without it, there are no capabilities to lock the character inside the main menu. Instead, the character will need to be changed to a different character when being initialized.

[EntityPlayer:ChangePlayerType](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#changeplayertype) will be utilized in order to change from your tainted character to your regular character. Using this function within [MC_POST_PLAYER_INIT](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_post_player_init) right as the player spawns will also grant the character any items that they may have on them as defined in their `players.xml` file. Depending on how your character is setup, you may need to make some additional adjustments to ensure this alternate way of starting as your regular character isn't any different from starting as them from the character selection menu.

```Lua
local mod = RegisterMod("My Mod", 1)

--PlayerTypes of the regular and tainted version of your character
local MY_CHAR = Isaac.GetPlayerTypeByName("My Character", false)
local MY_CHAR_TAINTED = Isaac.GetPlayerTypeByName("My Character", true)
local PLAYER_VARIANT_NORMAL = 0
--This is a stand-in for however you have your save data structured and where you keep your variable for tracking the tainted unlock.
local isUnlocked = false

--MC_POST_PLAYER_INIT passes the player being initialized.
function mod:LockTaintedOnInit(player)
	--Our tainted isn't unlocked yet!
	if player:GetPlayerType() == MY_CHAR_TAINTED and not isUnlocked then
		player:ChangePlayerType(MY_CHAR)
		--Insert other necessary adjustments here
	end
end

--MC_POST_PLAYER_INIT triggers whenever a player is initialized at run start, co-op spawn, run continue, or Genesis.
--PLAYER_VARIANT_NORMAL is inserted as the optional argument to ensure it runs for regular players and not co-op babies.
mod:AddCallback(ModCallback.MC_POST_PLAYER_INIT, mod.LockTaintedOnInit, PLAYER_VARIANT_NORMAL)
```

## Spawn the player body

The traditional method of unlocking a tainted character is by locating and touching their shaking body within the closet room of the Home floor. To start, check that you're entering the correct room with your character. The tainted character unlock method only looks at the first player using [Isaac.GetPlayer()](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getplayer).

You will be checking the first player's character multiple times throughout this tutorial. For convenience, create a function for it and return if the player is your character and has their tainted character locked.

```Lua
local game = Game()
--The Home floor has a static layout, so the room index of the closet should remain unchanged.
local CLOSET_ROOM_INDEX = 94

local function isFirstPlayerTaintedLocked()
	local player = Isaac.GetPlayer()
	local playerType = player:GetPlayerType()
	return playerType == MY_CHAR and not isUnlocked
end

function mod:SpawnTaintedOnClosetEnter()
	--Local variables for convenience.
	local room = game:GetRoom()
	local level = game:GetLevel()

	if level:GetStage() == LevelStage.STAGE8 --Home floor
		and level:GetCurrentRoomIndex() == CLOSET_ROOM_INDEX
		and room:IsFirstVisit() --We only need to spawn the body once.
		and isFirstPlayerTaintedLocked()
	then

	end
end

mod:AddCallback(ModCallbacks.MC_POST_NEW_ROOM, mod.SpawnTaintedOnClosetEnter)
```

:modding-repentogon: If using REPENTOGON, the `isFirstPlayerTaintedLocked` check should specifically involve the achievement attached to your character. As such, the function should be adjusted like so:

```Lua
--This is the achievement defined in achievements.xml and attached to your tainted character in players.xml
local TAINTED_ACHIEVEMENT = Isaac.GetAchievementIdByName("Tainted MyChar")
local persistGameData = Isaac.GetPersistentGameData()

local function isFirstPlayerTaintedLocked()
	local player = Isaac.GetPlayer()
	local playerType = player:GetPlayerType()
	return playerType == MY_CHAR and not persistGameData:Unlocked(TAINTED_ACHIEVEMENT)
end
```

Next, the room should be cleared of any extra entities and have the player body spawned. For all modded characters, tainted characters, and for vanilla characters that have their tainted variants unlocked, [Inner Child](https://bindingofisaacrebirth.wiki.gg/wiki/Inner_Child) will spawn if it is unlocked. If Inner Child is not unlocked, a shopkeeper will spawn instead. These are the only entities that need to be removed. For the player body, it is internally a slot machine (`EntityType.ENTITY_SLOT`) with a variant of `14`. :modding-repentogon: With REPENTOGON, there is a [SlotVariant](https://repentogon.com/enums/SlotVariant.html) enum with `14` being assigned to `SlotVariant.HOME_CLOSET_PLAYER`.

```Lua
local game = Game()
local CLOSET_ROOM_INDEX = 94
local SLOT_HOME_CLOSET_PLAYER = 14

function mod:SpawnTaintedOnClosetEnter()
	local room = game:GetRoom()
	local level = game:GetLevel()

	if level:GetStage() == LevelStage.STAGE8
		and level:GetCurrentRoomIndex() == CLOSET_ROOM_INDEX
		and room:IsFirstVisit()
		and isFirstPlayerTaintedLocked()
	then
		--Locate the first instance of an Inner Child collectible and Shopkeeper
		local innerChild = Isaac.FindByType(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_COLLECTIBLE, CollectibleType.COLLECTIBLE_INNER_CHILD)[1]
		local shopKeeper = Isaac.FindByType(EntityType.ENTITY_SHOPKEEPER)[1]

		--Remove Inner Child if found.
		if innerChild then
			innerChild:Remove()
		--A shopkeeper will only spawn if Inner Child isn't unlocked. If found, remove it.
		elseif shopKeeper then
			shopKeeper:Remove()
		end

		--Game():Spawn(EntityType, integer Variant, Vector Position, Vector Velocity, Entity SpawnerEntity, integer SubType, integer Seed)
		game:Spawn(EntityType.ENTITY_SLOT, SLOT_HOME_CLOSET_PLAYER, room:GetCenterPos(), Vector.Zero, nil, 0, Random())
	end
end

mod:AddCallback(ModCallbacks.MC_POST_NEW_ROOM, mod.SpawnTaintedOnClosetEnter)
```

## Update the player body sprite

The player body will attempt to take on the tainted appearance of the first player's current character. If no tainted is available, or when playing a modded character, it will spawn with the first player's own spritesheet. As such, the spritesheet must be manually updated to display the tainted's spritesheet.

### Non-REPENTOGON method

Without REPENTOGON, there are no callbacks for slot machines, so they must be manually searched for after they are spawned in and upon re-entering the room. The spritesheet to use must also be manually typed out as a string.

```Lua
local myCharTaintedSpritePath = "gfx/characters/costumes/character_mychar_b.png"

local function tryUpdateClosetIsaac()
	--We store this check before the FindByType loop as we only need to check it once.
	local firstPlayerTaintedLocked = isFirstPlayerTaintedLocked()
	--Search for all slot machines with the desired variant.
	for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_SLOT, SLOT_HOME_CLOSET_PLAYER)) do
		--Check that it just spawned and we should update it for our character.
		if ent.FrameCount == 0 and firstPlayerTaintedLocked then
			local sprite = ent:GetSprite()
			sprite:ReplaceSpritesheet(0, myCharTaintedSpritePath)
			sprite:LoadGraphics()
		end
	end
end

--Continuing with the mod:SpawnTaintedOnClosetEnter() function from earlier:
function mod:SpawnTaintedOnClosetEnter()
	--Don't actually type this part out, its just for reference.
	if ... then
		--Removed Inner Child or Shopkeeper
		--Spawned Closet player
	end
	--We want to update the body's sprite no matter what room it's located in, so this will activate on every MC_POST_NEW_ROOM.
	tryUpdateClosetIsaac()
end
```

### :modding-repentogon: REPENTOGON method

If you have REPENTOGON, you can update the spritesheet whenever the slot machine initializes on [MC_POST_SLOT_INIT](https://repentogon.com/enums/ModCallbacks.html#mc_post_slot_init). For additional convenience, you can automatically fetch your tainted counterpart's spritesheet using [EntityConfigPlayer](https://repentogon.com/EntityConfigPlayer.html).

```Lua
--Fetches the tainted spritesheet of the passed player.
local function getTaintedSpritesheet(player)
	--Current player's config.
	local playerConfig = player:GetEntityConfigPlayer()
	--Get their tainted's config. Keep in mind that on a tainted, it will return the regular counterpart.
	local taintedConfig = playerConfig:GetTaintedCounterpart()
	--If already a tainted character or has no tainted, returns the current player's spritesheet.
	if playerConfig:IsTainted() or not taintedConfig then
		--Returning early stops later code in this function from running.
		return playerConfig:GetSkinPath()
	end
	--Return the tainted character's spritesheet path.
	return taintedConfig:GetSkinPath()
end

function mod:OnClosetIsaacInit(slot)
	if isFirstPlayerTaintedLocked() then
		local sprite = slot:GetSprite()
		local player = Isaac.GetPlayer()
		local spritesheet = getTaintedSpritesheet(player)
		--Update the spritesheet. The last `true` here introduced by REPENTOGON will trigger `sprite:LoadGraphics()`.
		sprite:ReplaceSpritesheet(0, spritesheet, true)
	end
end

mod:AddCallback(ModCallbacks.MC_POST_SLOT_INIT, mod.OnClosetIsaacInit, SlotVariant.HOME_CLOSET_PLAYER)
```

## Triggering the unlock

When the player touches the player body, it will start playing the "PayPrize" animation. The unlock can be triggered when that animation finishes.

### Non-REPENTOGON method

The same method of finding the player body last time will be used here once more. This time, [MC_POST_UPDATE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_post_update) is used as the animation must be constantly checked for when it finishes. If you wish to show an achievement paper, that must be handled manually and is unable to be covered in this tutorial. Otherwise, you can use something such as [HUD:ShowItemText](https://wofsauge.github.io/IsaacDocs/rep/HUD.html#showitemtext) to communicate to the player that the character was unlocked.

```Lua
function mod:UnlockTaintedOnPayPrize()
	local firstPlayerTaintedLocked = isFirstPlayerTaintedLocked()
	for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_SLOT, SLOT_HOME_CLOSET_PLAYER)) do
		if firstPlayerTaintedLocked then
			local sprite = ent:GetSprite()
			if sprite:IsFinished("PayPrize") then
				isUnlocked = true
				game:GetHUD():ShowItemText("Tainted MyChar Unlocked!")
			end
		end
	end
end

mod:AddCallback(ModCallbacks.MC_POST_UPDATE, mod.UnlockTaintedOnPayPrize)
```

### :modding-repentogon: REPENTOGON method

REPENTOGON's [MC_POST_SLOT_UPDATE](https://repentogon.com/enums/ModCallbacks.html#mc_post_slot_update) will pass slot machines being updated, skipping the need for Isaac.FIndByType from the non-REPENTOGON method. You can unlock your registered achievement when the PayPrize animation finishes.

```Lua
function mod:UnlockTaintedOnPayPrize(slot)
	if isFirstPlayerTaintedLocked() then
		local sprite = slot:GetSprite()
		if sprite:IsFinished("PayPrize") then
			persistGameData:TryUnlock(TAINTED_ACHIEVEMENT)
		end
	end
end

mod:AddCallback(ModCallbacks.MC_POST_SLOT_UPDATE, mod.UnlockTaintedOnPayPrize, SlotVariant.HOME_CLOSET_PLAYER)
```

## Final code snippet

With that, your unlock is complete! For your convenience, the combined code for both non-REPENTOGON and REPENTOGON methods are available below:

### Non-REPENTOGON method

```Lua
local mod = RegisterMod("My Mod", 1)

local MY_CHAR = Isaac.GetPlayerTypeByName("My Character", false)
local MY_CHAR_TAINTED = Isaac.GetPlayerTypeByName("My Character", true)
local PLAYER_VARIANT_NORMAL = 0
local CLOSET_ROOM_INDEX = 94
local SLOT_HOME_CLOSET_PLAYER = 14
local isUnlocked = false
local myCharTaintedSpritePath = "gfx/characters/costumes/character_mychar_b.png"
local game = Game()

local function isFirstPlayerTaintedLocked()
	local player = Isaac.GetPlayer()
	local playerType = player:GetPlayerType()
	return playerType == MY_CHAR and not isUnlocked
end

local function tryUpdateClosetIsaac()
	local firstPlayerTaintedLocked = isFirstPlayerTaintedLocked()
	for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_SLOT, SLOT_HOME_CLOSET_PLAYER)) do
		if ent.FrameCount == 0 and firstPlayerTaintedLocked then
			local sprite = ent:GetSprite()
			sprite:ReplaceSpritesheet(0, myCharTaintedSpritePath)
			sprite:LoadGraphics()
		end
	end
end

function mod:LockTaintedOnInit(player)
	if player:GetPlayerType() == MY_CHAR_TAINTED and not isUnlocked then
		player:ChangePlayerType(MY_CHAR)
	end
end

mod:AddCallback(ModCallback.MC_POST_PLAYER_INIT, mod.LockTaintedOnInit, PLAYER_VARIANT_NORMAL)

function mod:SpawnTaintedOnClosetEnter()
	local room = game:GetRoom()
	local level = game:GetLevel()

	if level:GetStage() == LevelStage.STAGE8
		and level:GetCurrentRoomIndex() == CLOSET_ROOM_INDEX
		and room:IsFirstVisit()
		and isFirstPlayerTaintedLocked()
	then
		local innerChild = Isaac.FindByType(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_COLLECTIBLE, CollectibleType.COLLECTIBLE_INNER_CHILD)[1]
		local shopKeeper = Isaac.FindByType(EntityType.ENTITY_SHOPKEEPER)[1]

		if innerChild then
			innerChild:Remove()
		elseif shopKeeper then
			shopKeeper:Remove()
		end

		game:Spawn(EntityType.ENTITY_SLOT, SLOT_HOME_CLOSET_PLAYER, room:GetCenterPos(), Vector.Zero, nil, 0, Random())
	end
	tryUpdateClosetIsaac()
end

mod:AddCallback(ModCallbacks.MC_POST_NEW_ROOM, mod.SpawnTaintedOnClosetEnter)

function mod:UnlockTaintedOnPayPrize()
	local firstPlayerTaintedLocked = isFirstPlayerTaintedLocked()
	for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_SLOT, SLOT_HOME_CLOSET_PLAYER)) do
		if firstPlayerTaintedLocked then
			local sprite = ent:GetSprite()
			if sprite:IsFinished("PayPrize") then
				isUnlocked = true
				game:GetHUD():ShowItemText("Tainted MyChar Unlocked!")
			end
		end
	end
end

mod:AddCallback(ModCallbacks.MC_POST_UPDATE, mod.UnlockTaintedOnPayPrize)
```

### :modding-repentogon: REPENTOGON method

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_CHAR = Isaac.GetPlayerTypeByName("My Character", false)
local TAINTED_ACHIEVEMENT = Isaac.GetAchievementIdByName("Tainted MyChar")
local CLOSET_ROOM_INDEX = 94
local game = Game()
local persistGameData = Isaac.GetPersistentGameData()

local function isFirstPlayerTaintedLocked()
	local player = Isaac.GetPlayer()
	local playerType = player:GetPlayerType()
	return playerType == MY_CHAR and not persistGameData:Unlocked(TAINTED_ACHIEVEMENT)
end

local function getTaintedSpritesheet(player)
	local playerConfig = player:GetEntityConfigPlayer()
	local taintedConfig = playerConfig:GetTaintedCounterpart()
	if playerConfig:IsTainted() or not taintedConfig then
		return playerConfig:GetSkinPath()
	end
	return taintedConfig:GetSkinPath()
end

function mod:SpawnTaintedOnClosetEnter()
	local room = game:GetRoom()
	local level = game:GetLevel()

	if level:GetStage() == LevelStage.STAGE8
		and level:GetCurrentRoomIndex() == CLOSET_ROOM_INDEX
		and room:IsFirstVisit()
		and isFirstPlayerTaintedLocked()
	then
		local innerChild = Isaac.FindByType(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_COLLECTIBLE, CollectibleType.COLLECTIBLE_INNER_CHILD)[1]
		local shopKeeper = Isaac.FindByType(EntityType.ENTITY_SHOPKEEPER)[1]

		if innerChild then
			innerChild:Remove()
		elseif shopKeeper then
			shopKeeper:Remove()
		end

		game:Spawn(EntityType.ENTITY_SLOT, SlotVariant.HOME_CLOSET_PLAYER, room:GetCenterPos(), Vector.Zero, nil, 0, Random())
	end
end

mod:AddCallback(ModCallbacks.MC_POST_NEW_ROOM, mod.SpawnTaintedOnClosetEnter)

function mod:OnClosetIsaacInit(slot)
	if isFirstPlayerTaintedLocked() then
		local sprite = slot:GetSprite()
		local player = Isaac.GetPlayer()
		local spritesheet = getTaintedSpritesheet(player)
		--Update the spritesheet. The last `true` here introduced by REPENTOGON will trigger `sprite:LoadGraphics()`.
		sprite:ReplaceSpritesheet(0, spritesheet, true)
	end
end

mod:AddCallback(ModCallbacks.MC_POST_SLOT_INIT, mod.OnClosetIsaacInit, SlotVariant.HOME_CLOSET_PLAYER)

function mod:UnlockTaintedOnPayPrize(slot)
	if isFirstPlayerTaintedLocked() then
		local sprite = slot:GetSprite()
		if sprite:IsFinished("PayPrize") then
			persistGameData:TryUnlock(TAINTED_ACHIEVEMENT)
		end
	end
end

mod:AddCallback(ModCallbacks.MC_POST_SLOT_UPDATE, mod.UnlockTaintedOnPayPrize, SlotVariant.HOME_CLOSET_PLAYER)
```