---
article: Players and entities taking damage
authors: benevolusgoat
blurb: Learn how to use the "MC_ENTITY_TAKE_DMG" callback to respond to player or entity damage.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
---

The `MC_ENTITY_TAKE_DMG` callback has a number of useful applications for scenarios involving taking damage. This applies to enemies, players, and more. This tutorial will specifically focus on handling damage taken by the player.

## Callback breakdown
[MC_ENTITY_TAKE_DMG](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_entity_take_dmg) triggers when an entity takes damage, but **before the damage is subtracted from the entity's health**. It has 5 parameters, an optional argument, and a return type.

__Parameters__

- `ent` ([Entity](https://wofsauge.github.io/IsaacDocs/rep/Entity.html)): The entity taking damage.
- `amount` (`integer`): If the player took damage, represents how many half-hearts of damage they're taking. If an enemy, it will return a `float` for the exact amount of Hit Points to be subtracted from the enemy.
- `flags` ([DamageFlag](https://wofsauge.github.io/IsaacDocs/rep/enums/DamageFlag.html)): A bitflag containing extra information about the how the damage was dealt and how it should be handled. This flag can be `0` if there are no unique attributes.
- `source` ([EntityRef](https://wofsauge.github.io/IsaacDocs/rep/EntityRef.html)): The source of the damage. The entity that dealt the damage can be accessed through its `Entity` variable, **but this may return `nil`** such as if the damage came from spikes. In these instances, you can check the `Type` and `Variant` variables, where `Type` will be `0` and `Variant` will be the [`GridEntityType`](https://wofsauge.github.io/IsaacDocs/rep/enums/GridEntityType.html).
- `countdown` (`integer`): Will initiate a "damage countdown" if not already active. Any damage sources that deal damage with the `DamageFlag.DAMAGE_COUNTDOWN` flag, such as The Nail, Gamekid, and My Little Unicorn, will not deal damage while the countdown is active.

__Optional Argument__

The optional argument for this callback is `EntityType`. When specified, only entities with the matching type will trigger this callback.

```Lua
mod:AddCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg, EntityType.ENTITY_PLAYER) --Only the player taking damage will trigger this callback
```

__Return type__

You can return `false` to prevent the damage from from being taken by the entity. **Returning any value will stop the remaining callbacks from triggering.**

With :modding-repentogon: REPENTOGON, the callback's return functionality has been given an upgrade. You may return a table instead of a boolean and include any of the following fields to modify them:

- `Damage`: The amount of damage taken.
- `DamageFlags`: The corresponding DamageFlags associated with the damage being taken.
- `DamageCountdown`: The damage countdown set when the damage follows through.

In addition, instead of cancelling the remaining callbacks, it will pass these modified values along to them.

## Triggering events upon Isaac taking damage
Although this seems relatively simple, doing this incorrectly can make the difference of whether your code activates just once or in excessive amounts in a short period of time.

### Non-REPENTOGON
The only damage callback available is `MC_ENTITY_TAKE_DMG`. Mods may cancel damage inside this callback, cancelling future callbacks for that particular damage instance from running. If the damage goes through, Isaac will typically initiate invincibility frames. If the damage is cancelled, there's nothing stopping the callback from triggering a second time as soon as possible. If your mod were to trigger an event upon taking damage, and a mod that runs after yours cancels the damage, your code would trigger every single frame. As such, it is important to utilize `AddPriorityCallback` for situations where no "POST" callback exists.

[AddPriorityCallback](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#addprioritycallback) allows you to specify a [CallbackPriority](https://wofsauge.github.io/IsaacDocs/rep/enums/CallbackPriority.html), meaning you can guarantee it will run earlier or later than other callbacks that don't specify one. Using `CallbackPriority.LATE` means it will run after most other callbacks, thus can safely be used to trigger events upon taking damage.

```Lua
mod:AddPriorityCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, CallbackPriority.LATE, mod.OnEntityTakeDmg) --This will not run if a callback of default priority cancels the damage early
```

### :modding-repentogon: REPENTOGON
REPENTOGON adds `MC_POST_ENTITY_TAKE_DMG`, which has no return types but will only trigger after the damage has gone through fully, skipping the need for AddPriorityCallback.

## Modifying damage taken
It's common to want to modify the flags of any damage being taken to enemies or players. This process is relatively straightfoward.

### Non-REPENTOGON
The only way to change information about the damage being dealt is to cancel it and deal damage a second time with [Entity:TakeDamage](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#takedamage). Note that dealing damage within `MC_ENTITY_TAKE_DMG` the callback would typically result in an infinite loop. To avoid this, you can add the `DAMAGE_CLONES` damage flag in the `TakeDamage` function and then check for it. This DamageFlag is specifically used for avoiding infinite loops.

```Lua
function mod:OnEntityTakeDmg(ent, amount, flags, source, countdown)
	--Check for if the flag exists with the bitwise and operator.
	if ent:ToPlayer() and flags & DamageFlag.DAMAGE_CLONES == 0 then
		--Triggers MC_ENTITY_TAKE_DMG again with the same parameters, but reducing all damage taken to 1, like The Wafer.
		--Make sure to add the DAMAGE_CLONES flag to the bitfield of flags.
		player:TakeDamage(1, flags | DamageFlag.DAMAGE_CLONES, source, countdown)
		--Cancel the original damage, otherwise you'll be stacking damage taken!
		return false
	end
end

mod:AddCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg, EntityType.ENTITY_PLAYER)
```

### REPENTOGON
Referring back to the [Return type](taking_damage.md#callback-breakdown) section, you can return a table of values to modify attributes of the damage taken and it'll be passed along to the remaining callbacks.

```Lua
--All enemies take half the amount of damage they would've normally taken.
--This will be passed along to future callbacks.
function mod:OnEntityTakeDmg(ent, amount, flags, source, countdown)
	if ent:ToNPC() then
		return {Damage = amount * 0.5}
	end
end

mod:AddCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg)
```

## Safe/Fake player damage
Certain DamageFlags will prevent the player from losing devil or angel deal chance. The game uses this for things such as the curse room door. Detecting these sources can be seen within the `flags` parameter. You can also modify the damage taken to use these flags yourself.

- `DamageFlag.DAMAGE_NO_PENALTIES` will prevent any penalties to the player, such as lowering devil deal chance or resetting Perfection progress. This and `DamageFlag.DAMAGE_RED_HEARTS` will prevent rerolls from being trigered on [Tainted Eden](https://bindingofisaacrebirth.wiki.gg/wiki/Tainted_Eden).
- `DamageFlag.DAMAGE_FAKE` will not decrease Isaac's health, but can still trigger on-damage effects with the amount of damage that would've been taken. This is used by Dull Razor.

You can check, add, and remove bitflags using bitwise operations. You don't have to learn the exact details of how they work in order to utilize them, just remember what they do. The order in which they're positioned matters!
```Lua
--Returns `true` if flag2 is present inside flag1. Returns `0` otherwise.
flag1 & flag2 == flag2

--Combines flag1 and flag2. The "|" can be repeated for combining multiple flags.
flag3 = flag1 | flag2

--Removes flag2 from flag1
flag1 = flag1 & ~flag2
```
This code turns all player damage into no-penalty damage.
```Lua
function mod:OnEntityTakeDmg(ent, amount, flags, source, countdown)
	--Checks that the flag doesn't already have penalties removed.
	if ent:ToPlayer() and flags & DamageFlag.DAMAGE_NO_PENALTIES == 0 then
		--Deal damage with no penalties added to the original set of flags.
		player:TakeDamage(amount, flags | DamageFlag.DAMAGE_NO_PENALTIES, source, countdown)
		--Cancel the original damage.
		return false
	end
end

mod:AdCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg, EntityType.ENTITY_PLAYER)
```