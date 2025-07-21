---
article: Triggering an effect upon item pickup
authors: benevolusgoat
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Repentance
    - Repentance+
    - Lua
---

Having something happen when you finish picking up a collectible is an extremely simple, yet unsupported feature in the Isaac modding API.

## Introduction
Many items have some kind of effect upon picking it up for the first time, and not in other scenarios. This includes things like when Tainted Isaac swaps items back and forth on a pedestal, having Isaac's currently collected items rerolled through methods such as The D4, and more. These items will typically either grant health, coins, keys, or bombs directly, or spawn a pickup on the ground next to Isaac. Some of this may be achieved through variables in the `items.xml`, but others require Lua code. The vanilla Isaac modding API has no support for naturally triggering an effect upon the "first" pickup of a collectible, but this tutorial will help cover a workaround to get as close to identical behavior as the vanilla game as possible.

## Detecting you've picked up an item

### Non-REPENTOGON
The main goal of this tutorial is to assist with non-REPENTOGON users, given the lack of convenient tools to work with. Thankfully, there is a variable that can be accessed on players to see what item they're holding before it's put into their inventory. This variable is called [QueuedItem](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#queueditem), which gives access to a [QueuedItemData](https://wofsauge.github.io/IsaacDocs/rep/QueueItemData.html) object. Utilizing this variable, we can create a sort of on/off switch to know when we've picked up a specific item and when it's been completed to trigger our desired effect.

First, the essentials. You need your mod to add a callback and your item ID to know you're triggering your code for only your item:

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_ITEM = Isaac.GetItemIdByName("My Item")
local game = Game()
```

Next, the function you'll be nesting your code in and the callback to run it on. We'll be using `MC_POST_PEFFECT_UPDATE`, which is an "update callback" for the player. All entities have a callback that runs "on update", which means 30 times every second, the speed at which the game's internal logic runs. It is recommended to use this over `MC_POST_PLAYER_UPDATE` for most situations, as the 60 calls per second is usually unnecessary.

```Lua
function mod:OnPeffectUpdate(player)

end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

In order to store an on/off switch per-player, we will make use of GetData(). This is available for every entity in the game, which returns a table that's unique to that entity that can be used for arbitrary data. It's recommended to read on the page on [Entity Data](../concepts/entity_data.md) for detailed information. As an example, this code will trigger when your character suddenly changes into another character during a run.

```Lua
function mod:OnPeffectUpdate(player)
	--This will give us a table that will only ever exist for this player.
	local data = player:GetData()
	--Store your PlayerType here for ease of access.
	local playerType = player:GetPlayerType()

	--An "if not variable" check will check if it's equal to `nil`, meaning doesn't exist/hasn't been initialized yet, or `false`.
	--This will pass if we haven't set our "on" switch yet.
	if not data.TrackPlayerType then
		data.TrackPlayerType = playerType
	--This "elseif" statement will only ever run if our data DOES exist, thanks to the if check attached above it.
	elseif data.TrackPlayerType ~= playerType then
		--Execute code here
		--Be sure to update so that this doesn't trigger again right away!
		data.TrackPlayerType = playerType
	end
end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

`QueuedItem` has three variables: `Charge`, `Item`, and `Touched`.

- `Charge` is only relevant to active items. Gives you the current charge of the active item.
- `Item` is the [ItemConfigItem](https://wofsauge.github.io/IsaacDocs/rep/ItemConfig_Item.html) object that stores every piece of information on an item you could ask for, and as such is important for identifying what item it is. `QueuedItem` will always be available, but `Item` can return `nil` if Isaac is not holding any items above his head. **Remember that this can also have information on trinkets, not just collectibles**.
- `Touched` is the key to accomplishing the goal of this tutorial, indicating if this particular copy of the item has been picked up before or not. It will return `false` if not, `true` otherwise.

```Lua
function mod:OnPeffectUpdate(player)
	local data = player:GetData()
	--We store the variable here for ease of access; don't have to type it out every time you need it
	local heldItem = player.QueuedItem.Item

	--An "if variable" check will check if the variable exists (not `nil`) or is equal to `true`. This check will pass if Isaac is holding an item!
	if heldItem
		--Check the ID of the item matches your item's ID!
		and heldItem.ID == MY_ITEM
		--Important to check that it's a collectible, not a trinket! There's :IsTrinket() if you wish to check otherwise.
		and heldItem:IsCollectible()
		and not heldItem.Touched
		and not data.QueueMyItemPickup
	then
		data.QueueMyItemPickup = true
	--Check if we're no longer holding an item and we detected earlier we were holding our item.
	elseif not heldItem and data.QueueMyItemPickup then
		--Execute code here

		--Set it to `nil` or `false` so that this code doesn't trigger again right away.
		data.QueueMyItemPickup = nil
	end
end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

Finally, we can spawn something in!

```Lua
function mod:OnPeffectUpdate(player)
	local data = player:GetData()
	local heldItem = player.QueuedItem.Item

	if heldItem
		and heldItem.ID == MY_ITEM
		and heldItem:IsCollectible()
		and not data.QueueMyItemPickup
	then
		data.QueueMyItemPickup = true
	elseif not heldItem and data.QueueMyItemPickup then
		--Double-check the item actually made it into Isaac's inventory!
		if player:HasCollectible(MY_ITEM) then
			--Will find a free spot to spawn a pickup at least one tile away from Isaac. 40 = 1 tile.
			local pos = game:GetRoom():FindFreePickupSpawnPosition(player.Position, 40)
			--As we will be spawning pickups with Game():Spawn, it requires a seed to determine any RNG factors involved.
			--We can use GetCollectibleRNG as an easy way to get an RNG object to provide a seed for us that's specific to a collectible!
			local rng = player:GetCollectibleRNG(MY_ITEM)
			--Game():Spawn(Entity type, Entity variant, position, velocity, Entity who spawned it, Entity subtype, RNG seed)
			--The entity's SubType here is 0 to allow the game to determine what kind of heart pickup is spawned. Handy for hearts, keys, coins, bombs, trinkets, and collectibles.
			--rng:Next() will simultaneously advance the state of the RNG object (so it can return a different number next time its used)
			--and return the new seed it got from having its state advanced, perfect for nestling inside this function.
			Game():Spawn(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_HEART, pos, Vector.Zero, player, 0, rng:Next())
		end
		data.QueueMyItemPickup = nil
	end
end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

Unfortuntately, this method can only cover if Isaac has an item queued to be put into their inventory, such as collecting them from pedestals, and not from any methods that involve directly adding the item into Isaac's inventory, as the vanilla modding API does not provide any access to it. At the least, this covers the most common scenario of Isaac obtaining items.

### :modding-repentogon: REPENTOGON
The very first callback REPENTOGON added when in development was MC_POST_ADD_COLLECTIBLE. It will trigger whenever an item is added to Isaac's inventory, with a parameter to see if it's a first-time pickup or not. This is the perfect callback for the purposes of this tutorial.

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_ITEM = Isaac.GetItemIdByName("My Item")

--The variable we're looking to use is "firstTime", which tells us if it's a first-time pickup or not.
function mod:OnPostAddCollectible(itemID, charge, firstTime, activeSlot, varData, player)
	if firstTime then
		--Execute code here
	end
end

--The addition of the "MY_ITEM" argument for this callback means it will only ever trigger for your item.
mod:AddCallback(ModCallbacks.MC_POST_ADD_COLLECTIBLE, mod.OnPostAddCollectible, MY_ITEM)
```