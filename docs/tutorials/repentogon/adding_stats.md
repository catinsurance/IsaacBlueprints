---
article: Adding stats
authors: benevolusgoat
blurb: Learn how to add stats through various methods using REPENTOGON.
comments: true
tags:
    - Tutorial
    - Intermediate
    - Lua
    - XML
    - Repentance+
    - REPENTOGON
---

{% include-markdown "hidden/repentogon_notice.md" start="<!-- start -->" end="<!-- end -->" %}

Adding stats to the player within the vanilla API has been restricted to a single method: `MC_EVALUATE_CACHE`. This tutorial will cover the expanded accessibility to adding and adjusting stats on the player.

## Introduction

If a modder wishes to add stats to the player for any purpose without REPENTOGON, it would need to go through [MC_EVALUATE_CACHE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_evaluate_cache), after all vanilla calculations have been made. This includes character base stats and any damage and tears ups. Character stats are applied before all other stats, and damage and tears are split into three categories: Flat, regular, and multiplicative. [Damage](https://bindingofisaacrebirth.wiki.gg/wiki/Damage#Effective_Damage) and [tears](https://bindingofisaacrebirth.wiki.gg/wiki/Tears#Fire_Rate_Calculation) stats have unique formulas normally inaccessible in the modding API, and thus any modifications to those stats are impossible to be fully accurate to the vanilla game. REPENTOGON changes this with additions to both two XML files and Lua code.

## players.xml

First on the list of REPENTOGON's additions is the ability to set base stats to custom characters within the [players.xml](https://repentogon.com/xml/players.html) file. These stats will be applied before all other stat changes. Below is the list of available stats that can be applied:

???+ info "`players.xml` stat variables
	| Variable Name | Value | Comment |
	|:--|:--|:--|
	|speedmodifier|float|An inherent offset to the speed stat the character should start with. Base this offset off of Isaac's stats.|
	|firedelaymodifier|float|An inherent offset to the fire delay stat the character should start with. Base this offset off of Isaac's stats. The end result will not be *exactly* as expected due to various calculations done by the game, but as a baseline, `0.7` is equivalent to Sad Onion.|
	|damagemodifier|float|An inherent offset to the damage stat the character should start with. Base this offset off of Isaac's stats.|
	|rangemodifier|float|An inherent offset to the range stat the character should start with. Base this offset off of Isaac's stats.|
	|shotspeedmodifier|float|An inherent offset to the shot speed stat the character should start with. Base this offset off of Isaac's stats.|
	|luckmodifier|float|An inherent offset to the luck stat the character should start with. Base this offset off of Isaac's stats.|

???+ note "Adding negative tears stat"
	The implementation for the players.xml stat additions was based off of how Eden gets their stats. For fire delay, if the number is ever rolled as a negative, it gets multiplied by `0.686655` as to not be too severe.

Example XML entry:

```XML
<players root="gfx/characters/costumes/" portraitroot="gfx/ui/stage/" nameimageroot="gfx/ui/boss/">
	<player name="Gabriel" skin="character_gabriel.png" hp="6" skinColor="-1"
		  damagemodifier="-0.7" speedmodifier="0.25" luckmodifier="-0.8"
          nameimage="playername_gabriel.png" portrait="playerportrait_gabriel.png"
		  birthright="???"
	/>
</players>
```

## items.xml

Stats can now be directly added to items in the [items.xml](https://repentogon.com/xml/items.html) file without any Lua code. When the player obtains the item with the attached stats, it will be automatically applied to them. They're able to be added to all item types: passive, active, trinket, and REPENTOGON's null items. There are several variables for the damage and tears stat that are naturally integrated into the vanilla formula for their respective stats. This allows any added stats to respect many vanilla stat modifiers, such as multipliers.

Below are most of the available variables that can be added to any `items.xml` entries. There are more that are covered in the next section:

???+ info "`items.xml` stat variables
	???+ note "Automatic cache"
		When adding any of these variables, the associated `cache` for the stat is automatically added to the item, so there's no need to add it yourself.
	| Variable Name | Value | Comment |
	|:--|:--|:--|
	|tears|Offset's Isaac's tears stat before flat and mult modifiers. `0.7` is equivalent to Sad Onion.|
	|flattears|Offset's Isaac's tears stat additively; after regular tears ups but before multipliers. `0.5` is equivalent to Pisces.|
	|tearsmult|Offset's Isaac's tears stat multiplicatively; after regular and flat tears ups.|
	|damage|Offset's Isaac's damage stat before flat and mult modifiers stat. `1` is equivalent to Pentagram.|
	|flatdamage|Offset's Isaac's damage additively; after regular damage ups but before multipliers. `2` is equivalent to Curved Horn.|
	|damagemult|Offset's Isaac's damage mulltiplicatively; after regular and flat damage ups.|
	|shotspeed|Offset's Isaac's shot speed stat.|
	|speed|Offset's Isaac's speed stat.|
	|range|Offset's Isaac's range stat. `40` = +1 range.|
	|luck|Offset's Isaac's luck stat.|

Example XML entry:

```XML
<items gfxroot="gfx/items/" version="1">
    <passive id="1" name="Damage Potion" gfx="damage_potion_item.png" description="It smells like ashes and anger" quality="1" flatdamage="1" />
</items>
```

### Temporary Effects

You can choose to have stats only added when their respective [temporary effect](../general/temporary_effects.md) is present in addition to being added passively. To add one, prepend any of the variables in the previous section with `effect` (e.g. `effecttears`). Active items add a temporary effect of themselves when used, and thus can also add stats on use. Otherwise, they will need to be added manually through Lua code.

The following code will reduce Isaac's speed by 0.1 with each copy of the item and a permanent +1 luck for every bone heart collected.

`items.xml`:

```xml
<items gfxroot="gfx/items/" deathanm2="gfx/death_items.anm2" version="1">
    <passive id="1" name="Bone Charm" description="Skeletal Vigor" gfx="placeholder.png" quality="0" tags="summonable nolostbr"
        speed="-0.1" effectluck="1" persistent="true"
	/>
</items>
```

`main.lua`:

```Lua
local mod = RegisterMod("Bone Charm Mod", 1)
local BONE_CHARM = Isaac.GetItemIdByName("Bone Charm")

function mod:OnBoneHeartCollision(pickup, collider)
	local heartType = pickup.SubType
	local player = collider:ToPlayer()
	--Check the collider is a player, pickup is a bone heart, the player is able to collect bone hearts, and it was just collected.
	--IsDead() is true when a pickup gets collected, and the callback will not run more than once as the pickup's collision will be disabled automatically.
	if heartType == HeartSubType.HEART_BONE
		and player
		and player:CanPickBoneHearts()
		and pickup:IsDead()
	then
		--AddCollectibleEffect is an RGON-exclusive function that acts as a shortcut for adding collectible effects.
		player:AddCollectibleEffect(BONE_CHARM)
	end
end

--MC_POST_PICKUP_COLLISION is an RGON-exclusive callback; a POST version of MC_PRE_PICKUP_COLLISION.
mod:AddCallback(ModCallbacks.MC_POST_PICKUP_COLLISION, mod.OnBoneHeartCollision, PickupVariant.PICKUP_HEART)
```

## Advanced stat calculations

For any calculations that don't fit into what is provided by the XML additions, `MC_EVALUATE_CACHE` can still be used for most stats. For damage and tears specifically, REPENTOGON adds a new callback: [MC_EVALUATE_STAT](https://repentogon.com/enums/ModCallbacks.html#mc_evaluate_stat). The callback triggers on two different stages for each stat in the vanilla stat calculation, which can be seen through the [EvaluateStatStage](https://repentogon.com/enums/EvaluateStatStage.html) enum. Returning a value will override what's passed into the callback and pass it into the remaining callbacks.

Below is an example that adds a small damage up for each black heart Isaac has:

???+ note "XML Preference"
	The below example can still be achieved through XML additions. Due to how XML stats are implemented compared to the Lua callback, **it is strongly recommended that you use the new XML item stats features instead of this callback!

```Lua
local mod = RegisterMod("Demon Locket Mod", 1)

local DEMON_LOCKET = Isaac.GetItemIdByName("Demon Locket")
local DEMON_LOCKET_DAMAGE_UP = 0.7

function mod:DemonLocketDamageUp(player, statStage, value)
	if player:HasCollectible(DEMON_LOCKET) then
		return value + (player:GetBlackHearts() * DEMON_LOCKET_DAMAGE_UP)
	end
end

mod:AddCallback(ModCallbacks.MC_EVALUATE_STAT, mod.DemonLocketDamageUp, EvaluateStatStage.DAMAGE_UP)
```
