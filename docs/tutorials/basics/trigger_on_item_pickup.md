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

Having something happen when you finish picking up a collectible is a common yet unsupported feature in the Isaac modding API.

## Introduction
Many items have some kind of effect upon picking it up from a pedestal for the first time. These items will typically either grant health, coins, keys, or bombs directly, or spawn a pickup on the ground next to Isaac. Some of this may be achieved through variables in the `items.xml`, but others require Lua code. The vanilla Isaac modding API has no support for naturally triggering an effect upon the first pickup of a collectible from an item pedestal, but this tutorial will help cover a workaround to get as close to vanilla behavior as possible.

## Granting health or pickups through XML
If you're not looking to add any complex effect to your item on first pickup, you can make your item grant health or pickups through XML. Below is an example of a passive item that grants 5 bombs. You can find a list of attributes you can add [here](https://wofsauge.github.io/IsaacDocs/rep/xml/items.html).

```xml
<passive id="0" name="My Item" description="5 free bombs!" gfx="collectible_my_item.png" quality="0" bombs="5" />
```

## Detecting item pickup with Lua

### Without REPENTOGON
The main goal of this tutorial is to assist with non-REPENTOGON users, given the lack of convenient tools to work with. Thankfully, there is a variable that can be accessed on players to see what item they're holding before it's put into their inventory. This variable is called [`QueuedItem`](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#queueditem), which gives access to a [`QueuedItemData`](https://wofsauge.github.io/IsaacDocs/rep/QueueItemData.html) object. Utilizing this variable, we can know when we've picked up a specific item, and when the animation has finished so that we can trigger our desired effect.

Firstly, you'll need your item's ID:

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_ITEM = Isaac.GetItemIdByName("My Item")
local game = Game()
```

Next, create the function you'll be nesting your code in and the callback to run it on. We'll be using `MC_POST_PEFFECT_UPDATE`, which is an "update callback" that runs for every player. All entities have a callback that runs every time the entity is updated, which is 30 times every second. It is recommended to use this over `MC_POST_PLAYER_UPDATE` for most situations, as `MC_POST_PLAYER_UPDATE` runs 60 times per second.

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_ITEM = Isaac.GetItemIdByName("My Item")
local game = Game()

function mod:OnPeffectUpdate(player)
	--We'll put our code in here
end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

Now we'll use `QueuedItem` to read what item the player is holding above their head, and if they're holding any item at all. `QueuedItem` has three variables: `Charge`, `Item`, and `Touched`.

- `Charge` is only relevant to active items. It gives you the current charge of the active item.
- `Item` is the [ItemConfigItem](https://wofsauge.github.io/IsaacDocs/rep/ItemConfig_Item.html) object that stores every piece of information on an item you could ask for, and as such is important for identifying what item it is. `QueuedItem` will always be available, but `Item` can return `nil` if Isaac is not holding any items above his head. **Remember that this can also have information on trinkets, not just collectibles**.
- `Touched` is the key to accomplishing the goal of this tutorial, indicating if this particular copy of the item has been picked up before or not. It will return `false` if not, and `true` otherwise.

Additionally, we will make use of `GetData()`. This is a function available for every entity in the game which returns a table that's unique to that entity that can be used to store arbitrary data. It's recommended to read the article on [Entity Data](../concepts/entity_data.md) for detailed information about its many drawbacks.

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_ITEM = Isaac.GetItemIdByName("My Item")
local game = Game()

function mod:OnPeffectUpdate(player)
	local data = player:GetData()
	--We store the variable here for ease of access; don't have to type it out every time you need it
	local heldItem = player.QueuedItem.Item

	--An "if variable" check will check if the variable exists (not `nil`) or is equal to `true`. This check will pass if Isaac is holding an item!
	if heldItem
		--Check that the ID of the item matches your item's ID!
		and heldItem.ID == MY_ITEM
		--Important to check that it's a collectible, not a trinket! There's also :IsTrinket() if you wish to check the inverse.
		and heldItem:IsCollectible()
		and not heldItem.Touched
		-- Store that this was our item so that we can run the rest of code only if it is
		and not data.QueueMyItemPickup
	then
		data.QueueMyItemPickup = true
	--Check that we're no longer holding an item and for if the item was our item.
	elseif not heldItem and data.QueueMyItemPickup then
		--Execute code here

		--Set it to `nil` or `false` so that this code doesn't trigger again right away.
		data.QueueMyItemPickup = nil
	end
end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

Finally, we will spawn a random heart on first-time pickup.

```Lua
local mod = RegisterMod("My Mod", 1)
local MY_ITEM = Isaac.GetItemIdByName("My Item")
local game = Game()

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
			--rng:Next() will "advance" (change) the seed of the RNG object, returning that new seed
			game:Spawn(EntityType.ENTITY_PICKUP, PickupVariant.PICKUP_HEART, pos, Vector.Zero, player, 0, rng:Next())
		end

		data.QueueMyItemPickup = nil
	end
end

mod:AddCallback(ModCallbacks.MC_POST_PEFFECT_UPDATE, mod.OnPeffectUpdate)
```

???+ info "Note"
	This method will only cover if Isaac has an item queued to be put into his inventory, such as collecting them from pedestals, and not from any methods that involve directly adding the item into Isaac's inventory such as the debug console.

### :modding-repentogon: Using REPENTOGON
REPENTOGON adds a new callback named `MC_POST_ADD_COLLECTIBLE`. It will trigger whenever an item is added to Isaac's inventory through any means, with a parameter to see if it's a first-time pickup or not. This is the perfect callback for the purposes of this tutorial and simplifies our code tremendously.

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
--REPENTOGON only!
mod:AddCallback(ModCallbacks.MC_POST_ADD_COLLECTIBLE, mod.OnPostAddCollectible, MY_ITEM)
```