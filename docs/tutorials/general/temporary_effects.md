---
article: Temporary effects
authors: benevolusgoat
blurb: Learn about how to use the temporary effect system within Isaac.
comments: true
tags:
    - Tutorial
    - Intermediate
    - Repentance
    - Repentance+
    - REPENTOGON
    - Lua
---

TemporaryEffects are an excellent tool to utilize per-player effects, doubling as save data handled automatically by the game.

## Introduction

The [TemporaryEffects](https://wofsauge.github.io/IsaacDocs/rep/TemporaryEffects.html) class gives access to what is effectively an arbitrary boolean or integer unique to every player that is tied to a collectible, trinket, or null item that can be used for any desired purpose. They can be mistaken for "innate" collectibles or trinkets, which are invisible additions to Isaac's inventory of items, but they are entirely separate things.

A `TemporaryEffect` exists for every collectible, trinket, and null item automatically, but do not do anything on their own. While booleans and integers and other value types can be assigned to players just fine through [entity data](../concepts/entity_data.md), the unique attribute worth taking advantage of is their ability to be persistent across exiting and continuing the run, meaning **freely accessible run-lasting save data handled entirely by the game**.

## Viewing TemporaryEffects

Open the debug console using the tilde (`~`) key and type `debug 12`, then ENTER. This will enable the "Player Item Info" debug flag, which is helpful for displaying various information on the first player. This tutorial will be focusing on one specific section that lets you view TemporaryEffects active on the player.

Your screen may look like *this*. If it does, press TAB and it will switch to the correct view.

<p align="center">
  <img src="../../assets/temporary_effects/debug_12_1.jpg" alt="Incorrect debug 12 view" />
</p>

Below is the correct view for this tutorial:

<p align="center">
  <img src="../../assets/temporary_effects/debug_12_2.jpg" alt="Correct debug 12 view" />
</p>

When a TemporaryEffect is added to the player, it will show up on the "Player effects" list, as shown here:

<p align="center">
  <img src="../../assets/temporary_effects/temp_effect_preview.jpg" alt="Player effects view" />
</p>

## Understanding the types of TemporaryEffects

Below is a brief explanation on the different types of TemporaryEffects that you may see in this list and what they mean.

<p align="center">
  <img src="../../assets/temporary_effects/temp_effect_list.jpg" alt="Player effects list" />
</p>

- **<span style="color:rgba(255, 255, 255);">White</span>** text indicates a normal TemporaryEffect. It will be automatically removed upon exiting the current room.
- **<span style="color:rgba(255, 216, 0);">Yellow</span>** text indicates a persistent TemporaryEffect. It will not be removed when exiting the current room, persisting for the duration of the run, **including when exiting and continuing your run**.
- **<span style="color:rgba(0, 255, 255);">Cyan</span>** text with the `[R]` symbol indicates a TemporaryEffect applied to the room instead of the player. There is no visible difference between a persistent and non-persistent TemporaryEffect for rooms. They function identically to those added to players and exist for the purposes of easier accessibility, however this data cannot be accessed without :modding-repentogon: REPENTOGON.
- The name of the TemporaryEffect is self-explanatory: They display the name of their respective collectible, trinket, or null item.
- The number after the `x` is the amount of the TemporaryEffect on the player. This can be any positive or negative integer.
- The number after the `:` is the cooldown of the effect. It will have no special traits if undefined, thus showing `0`. If a cooldown above zero is defined, the number will be reduced by 1 at a rate of 30 per second (once per game tick). Upon reaching zero, the TemporaryEffect will be removed entirely, ignoring the amount of that item's effect that was present.

### Adding persistence or cooldowns to your items

For making your item's effect persistent or to add a cooldown, you will need to add the appropriate variable to your item's `items.xml` entry.

- `persistent` uses a boolean. Set to `true` to enable persistence for that item's TemporaryEffect.
- `cooldown` uses an integer. `30` is equivalent to one second.

```XML
<items gfxroot="gfx/items/" version="1">
	<passive id="1" name="My Item" gfx="my_item.png" description="It's my item!" persistent="true" cooldown="30" quality="0" />
</items>
```

## Adding TemporaryEffects

To add a TemporaryEffect to a player, first use `:GetEffects()` on an [`EntityPlayer`](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html) object. From here, you may use the `AddCollectibleEffect`, `AddTrinketEffect`, or `AddNullEffect` function. For this tutorial, we will be utilizing [AddCollectibleEffect](https://wofsauge.github.io/IsaacDocs/rep/TemporaryEffects.html#addcollectibleeffect).

This code will add "The Sad Onion" collectible effect on the first possible frame of the player spawning into the run.
```Lua
local mod = RegisterMod("My Effects Mod", 1)
local game = Game()

function mod:OnPeffectUpdate(player)
	if game:GetFrameCount() == 1 then
		local effects = player:GetEffects()
		--This will add 1 TemporaryEffect of Sad Onion. It will add the associated costume by default while the game manually accounts for increasing your tears stat if you have the effect.
		effects:AddCollectibleEffect(CollectibleType.COLLECTIBLE_SAD_ONION)
	end
end

--MC_POST_PLAYER_INIT is one of the earliest callbacks that run when first starting or continuing a run, so lots of other processes run after it.
--The step of Isaac's code that removes non-persistent TemporaryEffects upon entering a new room is one of them, meaning we must add it afterwards.
mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

### :modding-repentogon: REPENTOGON additions

REPENTOGON adds [AddCollectibleEffect](https://repentogon.com/EntityPlayer.html#addcollectibleeffect), [AddNullItemEffect](https://repentogon.com/EntityPlayer.html#addnullitemeffect) and [AddTrinketEffect](https://repentogon.com/EntityPlayer.html#addtrinketeffect) directly to the EntityPlayer class as a shortcut for their respective functions under `TemporaryEffects`.

It also adds the ability to access the `TemporaryEffects` object tied to rooms. It can be accessed through `Game():GetRoom():GetEffects()`.

## Utilizing TemporaryEffects

There are several ways to interact with TemporaryEffects using the various functions provided by the API. These are all accessed through the [TemporaryEffects](https://wofsauge.github.io/IsaacDocs/rep/TemporaryEffects.html) class.

- `HasCollectibleEffect`, `HasTrinketEffect`, and `HasNullEffect` return a boolean indicating whether or not the provided effect is present.
- `GetCollectibleEffectNum`, `GetTrinketEffectNum`, and `GetNullEffectNum` return an integer on how many stacks of the effect are active.
- `RemoveCollectibleEffect`, `RemoveTrinketEffect`, and `RemoveNullEffect` will remove 1 or more of the specified effect. Inputting `-1` will remove all instances of the effect.
- `GetCollectibleEffect`, `GetTrinketEffect`, and `GetNullEffect` are separate from their "Num" counterparts, returning a [TemporaryEffect](https://wofsauge.github.io/IsaacDocs/rep/TemporaryEffect.html) object. You can access three variables: `Cooldown`, `Count`, and `Item`. `Cooldown` returns an integer of the effect's cooldown (`0` if no cooldown is active), `Count` returns the same as `GetCollectibleEffectNum` and their other variants, and `Item` returns the `ItemConfigItem` object associated with the TemporaryEffect.
