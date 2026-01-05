---
article: Pocket Items - Pill Effects
authors: benevolusgoat
blurb: Learn how to create cusom pill effects.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - XML
    - Lua
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

## Introduction

Next to cards, runes, and objects, are the other major type of pocket item available in Isaac: Pills. They come in random colors with random pill effects each run, giving positive, negative, or neutral effects. This tutorial covers how to create a custom pill effect.

???+ bug "Custom Pill Colors"
	As a brief mention, unlike cards, custom pill colors are not naturally supported. If created through `entities2.xml`, they are unable to spawn naturally in a run and do not come assigned with a random pill effect, showing no text and not activating any effect when used. Other mods that add them force their own manual implementation.

## pocketitems.xml

Pill effects are created in the same way as cards; through the [pocketitems.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/pocketitems.html) file inside your mod's content folder. Unlike cards, they are marginally easier to create thanks to just being effects, while the visuals are handled by the randomized pill colors. They require the `pill` tag instead of `card`, which comes with its own unique set of variables.

???- info "`pill` tag variables"
	 ???+ note
		`name` is the only required variable. All others are optional.
	| Variable-Name | Possible Values | Description |
	|:--|:--|:--|
	|name|string|Name of the pill effect.|
	|description|string|Description of the pill effect (optional, used in I found pills).|
	|class|string|Possible values: [`3-`, `2-`, `1-`, `0`, `1+`, `2+`, `3+`]. Number indicates a Joke (`0`), Minor (`1`), Medium (`2`), or Major (`3`) effect. A `+` or `-` can be appended to note whether the pill is positive or negative, or excluded to denote neutral pills. Default = `0`.|
	|mimiccharge|int|Amount of charge the pill should take to mimic with [Placebo](https://bindingofisaacrebirth.wiki.gg/wiki/Placebo). Default = `4`|
	|announcer|int|Sound ID to play when the pill is used.|
	|announcer2|int|Sound ID to play when the pill is used as a horse pill.|
	|announcerdelay|int|Delay in frames between pill use and the sound provided being played.|
	|achievement|int|Ties the pill effect to a vanilla achievement.|
	|greedmode|bool|Is the pocketitem available in greedmode. Default = `true`.|

:modding-repentogon: REPENTOGON expands `pocketitems.xml` with the following variables for pill effects:

???- info ":modding-repentogon: REPENTOGON-exclusive `pill` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
    |achievement|int or string|Ties the pill-effect to be unlocked by an achievement. For modded ones, use the provided achievement name xml attribute(define one if it doesn't have one already).|

Below is an example of a basic pill effect entry:

```XML
<pocketitems>
    <pill name="Balls of Obsidian" class="2+"/>
</pocketitems>
```

## Lua code

With the pill effect created, only a short amount of Lua code is required to give it an effect. Create a `main.lua` file. Register your mod, use [Isaac.GetPillEffectByName](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#getpilleffectbyname) to fetch the ID of your pill, and create a single function attached to the [MC_USE_PILL](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_use_pill) callback. For this pill effect, add 2 black hearts:

```Lua
local mod = RegisterMod("My Mod", 1)

local BALLS_OF_OBSIDIAN = Isaac.GetPillEffectByName("Balls of Obsidian")

--MC_USE_PILL passes 3 arguments: The pill effect Id, the player using it, and UseFlags.
function mod:OnUsePill(pillEffect, player, useFlags)
    --1 unit is 1/2 a heart, so 4 = 2 black hearts.
    player:AddBlackHearts(4)
end

--MC_USE_PILL accepts an optional argument to only run for a specific pill.
mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnUsePill, BALLS_OF_OBSIDIAN)
```

When using the pill, it does not came with any animation by default. Positive pills are accompanied by [EntityPlayer:AnimateHappy](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html?#animatehappy), negative pills accompanied by [EntityPlayer:AnimateSad](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html?#animatesad), while neutral pills hold the pill above the player's head using [EntityPlayer:AnimatePill](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html?#animatepill), which requires a pill color.

### Pill Color On Use

Unfortuantely, `MC_USE_PILL` does not provide any method of detecting what pill color was involved in the pill effect's activation, so you will need to manually track what pill the player is holding before the pill is used.

Track the player's currently held pill color every player update using [MC_POST_PEFFECT_UPDATE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_post_peffect_update) and store the pill inside custom [entity data](../concepts/entity_data.md). This data can later be checked inside `MC_USE_PILL`.

???+ info ":modding-repentogon: REPENTOGON MC_USE_PILL PillColor Arg"
    With REPENTOGON, [MC_USE_PILL](https://repentogon.com/enums/ModCallbacks.html#mc_use_pill) passes a fourth argument that contains the pill color used in the pill effect use.

	```Lua
	function mod:OnUsePill(pillEffect, player, useFlags, pillColor)
		player:AddBlackHearts(4)
		player:AnimatePill(pillColor)
	end

	mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnUsePill, BALLS_OF_OBSIDIAN)
	```

```Lua
local mod = RegisterMod("My Mod", 1)

local BALLS_OF_OBSIDIAN = Isaac.GetPillEffectByName("Balls of Obsidian")

function mod:OnUsePill(pillEffect, player, useFlags)
	--Get the data that tracks the pill color, or if the data doesn't exist yet, get the currently held pill as a failsafe.
	local pillColor = data.MYMOD_HeldPillColor or player:GetPill(0)
	--A null pill color cannot be passed. Resort to another pill color by default.
	if pillColor == PillColor.PILL_NULL then
		pillColor = PillColor.PILL_BLUE_BLUE
	end
	player:AddBlackHearts(4)
	player:AnimatePill(pillColor)
end

--MC_USE_PILL accepts an optional argument to only run for a specific pill.
mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnUsePill, BALLS_OF_OBSIDIAN)

function mod:TrackPillColor(player)
	local data = player:GetData()
	--Will check 30 frames a second what the pill color in the player's primary pocket slot is.
	--If no pill is in the primary slot, will be pill `0`.
	data.MYMOD_HeldPillColor = player:GetPill(0)
end

mod:AddCallback(MC_POST_PEFFECT_UPDATE, mod.TrackPillColor)
```

### Horse Pills

Horse Pills were introduced in Repentance, being a powered up version of all existing pill effects. These are not new pill effects on their own, and as such do not require their own separate entry, but instead classified as pill colors.

To enact different behaviour for horse pills on the custom pill effect, you need to know if the used pill was from a horse pill color. They can be identified by their large number, being the normal pill color id + `PillColor.PILL_GIANT_FLAG`, being `1 << 11` which equates to `2048`. Unfortuantely, `MC_USE_PILL` does not provide any method of detecting what pill color was involved in the pill effect's activation, so you will need to manually track what pill the player is holding before the pill is used.

Track the player's currently held pill color every player update using [MC_POST_PEFFECT_UPDATE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_post_peffect_update) and store the pill inside custom [entity data](../concepts/entity_data.md). This data can later be checked inside `MC_USE_PILL`.

???+ info ":modding-repentogon: REPENTOGON MC_USE_PILL PillColor Arg"
    As with the previous section, REPENTOGON has the callback pass the pill color.

	```Lua
	function mod:OnUsePill(pillEffect, player, useFlags, pillColor)
		local blackHearts = 4
		local isHorse = pillColor & PillColor.PILL_GIANT_FLAG == PillColor.PILL_GIANT_FLAG
		if isHorse then
			blackHearts = blackHearts * 2
		end
		player:AddBlackHearts(blackHearts)
		player:AnimatePill(pillColor)
	end

	mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnUsePill, BALLS_OF_OBSIDIAN)
	```

```Lua
local mod = RegisterMod("My Mod", 1)

local BALLS_OF_OBSIDIAN = Isaac.GetPillEffectByName("Balls of Obsidian")

--Pass the player and the UseFlags from the pill activation.
--Credit to this code goes to Xalum, who developed this for Fiend Folio.
local function isUsingHorsePill(player, useFlags)
	local data = player:GetData()
	--Get the data that tracks the pill color, or if the data doesn't exist yet, get the currently held pill as a failsafe.
	local pillColor = data.MYMOD_HeldPillColor or player:GetPill(0)

	local holdingHorsePill = pillColour & PillColor.PILL_GIANT_FLAG == PillColor.PILL_GIANT_FLAG
	--We cannot know the pill color if it was activated through Echo Chamber.
	--Echo Chamber is the only effect that uses the USE_NOHUD flag for pills, so disable usage if the flag is active.
	local proccedByEchoChamber = flags & UseFlag.USE_NOHUD == UseFlag.USE_NOHUD

	return holdingHorsePill and not proccedByEchoChamber
end

function mod:OnUsePill(pillEffect, player, useFlags)
	local blackHearts = 4
	local isHorse = isUsingHorsePill(player, useFlags)
	if isHorse then
		blackHearts = blackHearts * 2
	end
	player:AddBlackHearts(blackHearts)
end

--MC_USE_PILL accepts an optional argument to only run for a specific pill.
mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnUsePill, BALLS_OF_OBSIDIAN)

function mod:TrackPillColor(player)
	local data = player:GetData()
	--Will check 30 frames a second what the pill color in the player's primary pocket slot is.
	data.MYMOD_HeldPillColor = player:GetPill(0)
end

mod:AddCallback(MC_POST_PEFFECT_UPDATE, mod.TrackPillColor)
```

### Positive/Negative Effect Counterparts

With items like [PHD](https://bindingofisaacrebirth.wiki.gg/wiki/PHD) and [False PHD](https://bindingofisaacrebirth.wiki.gg/wiki/False_PHD), pill effects may have an assigned counterpart and be forced to turn into said counterparts. For example, False PHD will turn a Tears Up Pill into a Tears Down Pill, while PHD will do the opposite. To achieve this with custom pills, [MC_GET_PILL_EFFECT](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_get_pill_effect) can be used. The player will need to be checked if they meet the requirements for positive or negative pills. Without REPENTOGON, the callback does not pass the player involved. This means the only option is to check **all players** to affect what to change the pill effect into.

For this example, a counterpart will be created to `Balls of Obsidian` named `Balls of Paper`. Define the new pill effect and a duplicate callback to give it functionality:

```XML
<pocketitems>
    <pill name="Balls of Obsidian" class="2+"/>
    <pill name="Balls of Paper" class="2-"/>
</pocketitems>
```

```Lua
local mod = RegisterMod("My Mod", 1)

local BALLS_OF_OBSIDIAN = Isaac.GetPillEffectByName("Balls of Obsidian")
local BALLS_OF_PAPER = Isaac.GetPillEffectByName("Balls of Paper")

function mod:OnPillUse(pillEffect, player, useFlags)
	if pillEffect == BALLS_OF_OBSIDIAN then
		player:AddBlackHearts(4)
		player:AnimateHappy()
	elseif pillEffect == BALLS_OF_PAPER then
		player:AddBlackHearts(-4)
		player:AnimateSad()
	end
end
mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnPillUse, BALLS_OF_OBSIDIAN)
mod:AddCallback(ModCallbacks.MC_USE_PILL, mod.OnPillUse, BALLS_OF_PAPER)
```

Next, setup a function that will return if the player has positive or negative pill effects forced.

The Non-REPENTOGON method involves looping through all players with [Isaac.FindByType](https://wofsauge.github.io/IsaacDocs/rep/Isaac.html#findbytype):

```Lua
--Define an enum for readability
local PillEffectSubClass = {
	NEUTRAL = 0,
	POSITIVE = 1,
	NEGATIVE = 2
}

local function getPillEffectSubClass()
	local players = Isaac.FindByType(EntityType.ENTITY_PLAYER)
	local shouldBePositive = false
	local shouldBeNegative = false

	--Loop through all players in the room using Isaac.FindByType.
	for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_PLAYER)) do
		--It passes an Entity object by default. Cast to EntityPlayer to use player-related functions.
		local player = ent:ToPlayer()
		--All vanilla methods that enforce positive pills.
		if player:HasCollectible(CollectibleType.COLLECTIBLE_LUCKY_FOOT)
			or player:HasCollectible(CollectibleType.COLLECTIBLE_VIRGO)
			or player:HasCollectible(CollectibleType.COLLECTIBLE_PHD)
			or player:GetZodiacEffect() == CollectibleType.COLLECTIBLE_VIRGO
		then
			shouldBePositive = true
		end
		--False PHD is the only item that enforces negative pills.
		if player:HasCollectible(CollectibleType.COLLECTIBLE_FALSE_PHD) then
			shouldBeNegative = true
		end
		--Having both effects cancel each other out. End the loop early and return a value.
		if shouldBePositive and shouldBeNegative then
			return PillEffectSubClass.NEUTRAL
		end
	end

	if shouldBePositive then
		return PillEffectSubClass.POSITIVE
	elseif shouldBeNegative then
		return PillEffectSubClass.NEGATIVE
	else
		return PillEffectSubClass.NEUTRAL
	end
end
```

:modding-repentogon: The REPENTOGON method involves already having the player, so the function can be shortened:

```Lua
--Define an enum for readability
local PillEffectSubClass = {
	NEUTRAL = 0,
	POSITIVE = 1,
	NEGATIVE = 2
}

local function getPillEffectSubClass(player)
	local shouldBePositive = false
	local shouldBeNegative = false

	--All vanilla methods that enforce positive pills.
	if player:HasCollectible(CollectibleType.COLLECTIBLE_LUCKY_FOOT)
		or player:HasCollectible(CollectibleType.COLLECTIBLE_VIRGO)
		or player:HasCollectible(CollectibleType.COLLECTIBLE_PHD)
		or player:GetZodiacEffect() == CollectibleType.COLLECTIBLE_VIRGO
	then
		shouldBePositive = true
	end
	--False PHD is the only item that enforces negative pills.
	if player:HasCollectible(CollectibleType.COLLECTIBLE_FALSE_PHD) then
		shouldBeNegative = true
	end

	--Having both effects cancel each other out.
	if shouldBePositive and shouldBeNegative then
		return PillEffectSubClass.NEUTRAL
	elseif shouldBePositive then
		return PillEffectSubClass.POSITIVE
	elseif shouldBeNegative then
		return PillEffectSubClass.NEGATIVE
	else
		return PillEffectSubClass.NEUTRAL
	end
end
```

Finally, setup the `MC_GET_PILL_EFFECT` function:

```Lua
function mod:AssignPillCounterparts(pillEffect, pillColor)
	--If using REPENTOGON, you would define the third argument above as "player" and pass it into the function below.
	local pillSubClass = getPillEffectSubClass()
	if pillEffect == BALLS_OF_OBSIDIAN and pillSubClass == PillEffectSubClass.NEGATIVE then
		return BALLS_OF_PAPER
	elseif pillEffect == BALLS_OF_PAPER and pillSubClass == PillEffectSubClass.POSITIVE
		return BALLS_OF_OBSIDIAN
	end
end

mod:AddCallback(ModCallbacks.MC_GET_PILL_EFFECT, mod.AssignPillCounterparts, BALLS_OF_OBSIDIAN)
mod:AddCallback(ModCallbacks.MC_GET_PILL_EFFECT, mod.AssignPillCounterparts, BALLS_OF_PAPER)
```

### Announcer Voiceline

As with cards, pill effects also have announcer voicelines. Please see the [previous article](pocket_item_cards.md#announcer-voiceline) on cards for the exact details on creating a system for announcer voicelines. The only notable difference is that horse pills have their own unique announcer voiceline, so a second sound will need to be created that will only play if you know the used pill is from a horse pill.