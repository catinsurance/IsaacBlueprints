---
article: Taking damage
authors: benevolusgoat
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
---

The `MC_ENTITY_TAKE_DMG` callback has a number of useful applications for scenarios involving taking damage, namely as the player, so long as it is done correctly.

## Callback breakdown
[MC_ENTITY_TAKE_DMG](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_entity_take_dmg) triggers when an entity takes damage, but **before the damage is subtracted from the entity's health**. It has 5 parameters, an optional arg, and a return type.

__Parameters__

- ent ([Entity](https://wofsauge.github.io/IsaacDocs/rep/Entity.html)): The entity taking damage
- amount (`integer`): If the player took damage, represents how many half-hearts of damage they're taking. If an enemy, it will return a `float` for the exact amount of Hit Points to be subtracted from the enemy.
- flags ([DamageFlag](https://wofsauge.github.io/IsaacDocs/rep/enums/DamageFlag.html)): A bitflag containing information about what type of damage it is and may come with certain effects if the damage goes through.
- source ([EntityRef](https://wofsauge.github.io/IsaacDocs/rep/EntityRef.html)): The source of the damage. The entity that dealt the damage can be accessed through its `Entity` variable, **but this may return `nil`** such as if the damage came from spikes.
- countdown (`integer`): Will initiate a "damage countdown" if not already active. Any damage sources that deal damage with the `DamageFlag.DAMAGE_COUNTDOWN` flag, such as The Nail, Gamekid, and My Little Unicorn, will not deal damage while the countdown is active

__Optional Arg__

The Optional Arg for this callback is `EntityType`. When specified, only entities with the matching type will trigger this callback.

```Lua
mod:AddCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg, EntityType.ENTITY_PLAYER) --Only the player taking damage will trigger this callback
```

__Return type__

You can return `false` to prevent the damage from finalizing, thus the entity is left with no damage taken. **Returning any value will stop the remaining callbacks from triggering**.

With :modding-repentogon: REPENTOGON, the callback's return functionality has been given an upgrade. You may return a table instead of a boolean and include any of the following fields to modify them:
- `Damage`: The amount of damage taken.
- `DamageFlags`: The corresponding DamageFlags associated with the damage being taken.
- `DamageCountdown`: The damage countdown set when the damage follows through.
In addition, instead of cancelling the remaining callbacks, it will pass these modified values along to them.

## Triggering events upon Isaac taking damage
Although this seems relatively simple, doing this incorrectly can make the difference of whether your code activates just once or in excessive amounts in a short period of time.

### Non-REPENTOGON
The only damage callback available is `MC_ENTITY_TAKE_DMG`. Mods may cancel damage inside this callback, cancelling future callbacks for that particular damage instance from running. If the damage goes through, Isaac will typically initiate i-frames, stopping the callback from running altogether while active. If the damage is cancelled, there's nothing stopping the callback from triggering a second time as soon as possible. If your mod were to trigger an event upon taking damage and a mod that runs after yours cancels the damage, your code would trigger every single frame. As such, it is important to utilize `AddPriorityCallback` for situations where a "PRE" callback exists, but no "POST" callback.

[AddPriorityCallback](https://wofsauge.github.io/IsaacDocs/rep/ModReference.html#addprioritycallback) allows you to specify a [CallbackPriority](https://wofsauge.github.io/IsaacDocs/rep/enums/CallbackPriority.html), meaning you can guarantee it will run earlier or later than other callbacks that don't specify one. Using `CallbackPriority.LATE` means it will run after most other callbacks, thus can safely be used to trigger events upon taking damage.

```Lua
mod:AddPriorityCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, CallbackPriority.LATE, mod.OnEntityTakeDmg) --This will not run if a callback of default priority cancels the damage early
```

### :modding-repentogon: REPENTOGON
REPENTOGON adds `MC_POST_ENTITY_TAKE_DMG`, which has no return types but will only trigger after the damage has gone through fully, skipping the need for AddPriorityCallback.

## Modifying damage taken
It's common to want to modify attributes of any damage being taken to enemies or players. Thankfully, it's a relatively straightforward process.

### Non-REPENTOGON
The only way to change information about the damage being dealt is to cancel it and deal damage a second time with [Entity:TakeDamage](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#takedamage). Dealing damage inside a callback that deals damage would result in an infinite loop, so it's important to set boolean switches to prevent this.

```Lua
function mod:OnEntityTakeDmg(ent, amount, flags, source, countdown)
	local data = ent:GetData()
	--Setting a check ahead of time so that we know this won't trigger a second time
	if ent:ToPlayer() and not data.Dealt1PlayerDamage then
		--Set our boolean to true
		data.Dealt1PlayerDamage = true
		--Triggers MC_ENTITY_TAKE_DMG again with the same parameters, but reducing all damage taken to 1, like The Wafer
		player:TakeDamage(1, flags, source, countdown)
		--Setting it back to false so this code can trigger again in the future
		data.Dealt1PlayerDamage = false
		--Cancel the original damage, otherwise we'd be stacking damage taken!
		return false
	end
end

mod:AdCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg, EntityType.ENTITY_PLAYER)
```

### REPENTOGON
Referring back to the [Return type](taking_damage.md#callback_breakdown) section, you can return a table of values to modify attributes of the damage taken and it'll be passed along to the remaining callbacks.

```Lua
--All enemies take half the amount of damage they would've normally taken.
--This will be passed along to future callbacks.
function mod:OnEntityTakeDmg(ent, amount, flags, source, countdown)
	if ent:ToNPC() then
		return {Damage = amount * 0.5}
	end
end

mod:AdCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg)
```

## Safe/Fake player damage
It's critical game knowledge to know what sources of damage can punish you, such as lowering your devil deal chance, and what won't. It's even more crucial that modders keep these sources in mind as to correctly handle player damage for any intended purposes.

How to detect these sources can be seen in the `flags` parameter containing bitflags using the `DamageFlag` enumeration. You can also modify the damage taken to use these flags yourself.

- `DamageFlag.DAMAGE_NO_PENALTIES` will prevent any penalties to the player, such as lowering devil deal chance or resetting Perfection progress. This and `DamageFlag.DAMAGE_RED_HEARTS` will NOT trigger rerolls on [Tainted Eden](https://bindingofisaacrebirth.wiki.gg/wiki/Tainted_Eden).
- `DamageFlag.DAMAGE_FAKE` will not decrease Isaac's health, but can still trigger on-damage effects with the amount of damage that would've been taken (i.e. taking damage with an amount of 1 but with this flag active means Isaac will take 1 point of damage without any health reduction).

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
	local data = ent:GetData()
	--Checks that the flag doesn't already have penalties removed
	if ent:ToPlayer() and flags & DamageFlag.DAMAGE_NO_PENALTIES == 0 then
		--Deal damage with no penalties added to the original set of flags
		player:TakeDamage(amount, flags | DamageFlag.DAMAGE_NO_PENALTIES, source, countdown)
		--Cancel the original damage
		return false
	end
end

mod:AdCallback(ModCallbacks.MC_ENTITY_TAKE_DMG, mod.OnEntityTakeDmg, EntityType.ENTITY_PLAYER)
```