---
article: Familiars
authors: benevolusgoat
blurb: Learn how to create custom familiars.
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - Lua
    - XML
    - Repentance
    - Repentance+
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Familiars are small companions Isaac obtains, normally through passive items, that follow and assist Isaac. This tutorial will cover the basics of creating a familiar that can shoot tears.

## Video tutorial

[![Familiars | Youtube Tutorial](https://img.youtube.com/vi/kJoTeVR8tAA/0.jpg)](https://youtu.be/kJoTeVR8tAA "Video tutorial")

## Creating your familiar

Creating your familiar involves entries within two different files: [items.xml](passive_item.md) and [entities2.xml](entity_basics.md). Adjustments are needed to cater these entries towards familiars specifically.

### items.xml

When defining your item, instead of the `passive` tag, use `familiar`. This will categorize the item as a familiar item. Normally, an item that summons a familiar should have `cache="familiars"` present, but items that are defined as a familiar item will automatically handle this.

This `items.xml` entry will add an item named "Friend Frankie", a quality 1 item with four tags present.

- `baby` will have the item contribute to the [Conjoined](https://bindingofisaacrebirth.wiki.gg/wiki/Conjoined) transformation.
- `offensive` makes this item available to Tainted Lost.
- `summonable` allows the item to be summoned by [Lemegeton](https://bindingofisaacrebirth.wiki.gg/wiki/Lemegeton).
- `monstermanual` allows the item's familiar to be summoned by [Monster Manual](https://bindingofisaacrebirth.wiki.gg/wiki/Monster_Manual).

```XML
<items gfxroot="gfx/items/" version="1">
	<familiar name="Friend Frankie" description="Friends 'till the end" quality="1" tags="baby offensive summonable monstermanual" />
</items>
```

### entities2.xml

In conjunction with your item, the familiar must exist as an entity. The `id` variable must be equal to `3` to define it as a familiar.

This `entities2.xml` entry will add a familiar entity, aptly named "Friend Frankie", with some simple attributes. Note that the `items.xml` entry and the `entities2.xml` entry do not need to have the same name; it is only for convenience.

For the variant, it can be any arbitrary value not taken up by an existing vanilla familiar. Available variants are `131`-`199`, `244`-`899`, and `901`-`4095`. You can also omit the variant entirely to have the game auto-assign an available variant.

???+ note "Auto-assigning variant"
	If your familiar variant is automatically assigned by omitting it from your entry, and your familiar is the first to load before other mods that do the same, it will be assigned a variant of `0` as there is no vanilla familiar with that variant. When the game attempts to spawn an entity with an invalid variant, it will try to spawn it with a variant of `0`. If no entity of that variant exists, the game will crash.

	Other mods that may attempt to spawn an invalid familiar will spawn your familiar instead of causing a game crash, which may lead to unwanted bug reports, but there is no technical downside to this method.

```XML
<entities anm2root="gfx/" version="5">
    <entity name="Friend Frankie" id="3" variant="508" anm2path="familiar_friend_frankie.anm2"
    tags="cansacrifice" shadowSize="11" friction="1" />
</entities>
```

### Familiar relevant tags

The `tags` variable has several options available that are exclusive to or relevant for familiars.

???- info "`tags` list"
	| Tag-Name | Suffix |
	|:--|:--|
	| cansacrifice | Marks familiars that [Sacrificial Altar](https://bindingofisaacrebirth.wiki.gg/wiki/Sacrificial_Altar) can be used on. |
	| fly | Increases familiar size and damage if their player has Hive Mind, similarly to [BFFS!](https://bindingofisaacrebirth.wiki.gg/wiki/BFFS!). |
	| spider | Increases familiar size and damage if their player has Hive Mind, similarly to [BFFS!](https://bindingofisaacrebirth.wiki.gg/wiki/BFFS!). |

:modding-repentogon: REPENTOGON also has a `customtags` variable intended to expand upon the `tags` list.

???- info "`customtags` list"
	| Tag-Name | Suffix |
	|:--|:--|
	| familiarignorebffs | The [BFFS!](https://bindingofisaacrebirth.wiki.gg/wiki/BFFS!) item will no longer affect this familiar. |
	| familiarcantakedamage | Familiars with `baseHP` above `0` will be able to take damage and die, such as from enemy contact, lasers or explosions. Note that they will only take damage from projectiles if they also have the `familiarblockprojectiles` tag. |
	| familiarblockprojectiles | The familiar automatically destroys enemy projectiles on contact. |
	| nocharm | Makes the familiar not be charmed by [Siren](https://bindingofisaacrebirth.wiki.gg/wiki/The_Siren). |

## Linking the familiar to the item

Although you have created your item and your entity, there is nothing that automatically links the two together. This must be done manually through Lua code through the use of the [MC_EVALUATE_CACHE](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_evaluate_cache) callback.

Firstly, set up the `main.lua` file at the root of your mod folder, and fetch the necessary variables needed and the callback.

```Lua
local mod = RegisterMod("My Mod", 1)

--Obtain the ID automatically generated by the game for both your item and your entity.
--You will be storing other variables here that do not change, named constants, later on.
local FRIEND_FRANKIE_ID = Isaac.GetItemIdByName("Friend Frankie")
local FRIEND_FRANKIE_FAMILIAR = Isaac.GetEntityVariantByName("Friend Frankie")

---The player who's cache is being evaluated is the first argument passed into the function.
function mod:EvaluateCache(player)

end

--MC_EVALUATE_CACHE allows an optional argument where you can define a CacheFlag, which will make the callback only run for that flag.
--In this instance, the function will only ever run for the CACHE_FAMILIARS CacheFlag.
mod:AddCallback(ModCallbacks.MC_EVALUATE_CACHE, mod.EvaluateCache, CacheFlag.CACHE_FAMILIARS)
```

Checking what familiars should spawn, adding familiars, or removing them is all handled through the [EntityPlayer:CheckFamiliar](https://wofsauge.github.io/IsaacDocs/rep/EntityPlayer.html#checkfamiliar) function. It is not necessarily required, but is convenient for most use-cases. The function takes the following arguments:

1. What familiar variant to spawn.
2. How many familiars are expected to be present.
3. An [RNG](https://wofsauge.github.io/IsaacDocs/rep/RNG.html) object for determining its `InitSeed`.
4. An [ItemConfigItem](https://wofsauge.github.io/IsaacDocs/rep/ItemConfig_Item.html) object for linking an item to the spawned familiar. This is necessary for [Sacrifical Altar](https://bindingofisaacrebirth.wiki.gg/wiki/Sacrificial_Altar) to function properly, but is technically optional.
5. An integer to spawn a specific `SubType` of familiar. This is optional, and by default spawns familiars with a subtype of `0`.

The familiar variant is already stored earlier in the lua file. For the number of familiars to spawn, this can be any number you'd like, but the most common example is how many copies of the item Isaac has combined with the number of [TemporaryEffects](../general/temporary_effects.md) of the item. The purpose of the latter is for items such as [Box of Friends](https://bindingofisaacrebirth.wiki.gg/wiki/Box_of_Friends), where it will add a `TemporaryEffect` of your familiar item to spawn a copy of your familiar without needing to own the item.

???+ bug "CheckFamiliar and RNG"
	The `RNG` object passed into `CheckFamiliar` is used for the familiar's `InitSeed`, which is often used by modders as a unique identifier for each individual familiar.

	`CheckFamiliar` is bugged in that it will get new seeds from the RNG object provided *without* actually changing the object. This means that if you use `CheckFamiliar` to spawn 3 new familiars at once, they will all have different seeds, but the RNG object will still have the seed it started with. However, if you use `CheckFamiliar` to spawn one familiar 3 separate times (like Box of Friends), they'll all have the same seed.

	A simple way to fix this is to either advance the RNG object manually for each familiar spawned, or to just provide a unique RNG object to `CheckFamiliar` every time you use it. This tutorial will use the latter solution for simplicity.

```Lua
local mod = RegisterMod("My Mod", 1)

local FRIEND_FRANKIE_ID = Isaac.GetItemIdByName("Friend Frankie")
local FRIEND_FRANKIE_FAMILIAR = Isaac.GetEntityVariantByName("Friend Frankie")
--Storing constant variable here for convenience.
local RNG_SHIFT_INDEX = 35
--Access the ItemConfig class.
local itemConfig = Isaac.GetItemConfig()
--Obtain the ItemConfigItem object for our item.
local CONFIG_FRANKIE = itemConfig:GetCollectible(FRIEND_FRANKIE_ID)

function mod:EvaluateCache(player)
	--Access the TemporaryEffects class.
	local effects = player:GetEffects()
	--Get how many familiars should be spawned.
	local count = player:GetCollectibleNum(FRIEND_FRANKIE_ID) + effects:GetCollectibleEffectNum(FRIEND_FRANKIE_ID)
	--Create a new RNG object.
	local rng = RNG()
	--Get a random seed via Random(), which returns any number from 0 to 2^32, inclusive.
	--RNG objects with a seed of 0 will crash the game, so math.max is used to ensure it's always at least 1 or above.
	local seed = math.max(Random(), 1)
	--When setting a seed, you must set the "Shift Index", which essentially determines the sequence of numbers the RNG object will have.
	--The most commonly used Shift Index in vanilla was revealed to be 35 by one of the game's developers.
	rng:SetSeed(seed, RNG_SHIFT_INDEX)

	player:CheckFamiliar(FRIEND_FRANKIE_FAMILIAR, count, rng, CONFIG_FRANKIE)
end

mod:AddCallback(ModCallbacks.MC_EVALUATE_CACHE, mod.EvaluateCache, CacheFlag.CACHE_FAMILIARS)
```

With that, your familiar should now spawn when collecting your item and through other scenarios such as Box of Friends!

## Familiar variants

Your familiar exists, but at the current moment will do nothing when spawned in. Your familiar can do anything you'd like it to, but there are several definitive types of familiars that you can create with the modding API.

<p align="center">
  <img src="../../assets/familiars/follower_spawn.png" alt="The familiar as seen in-game" />
</p>

### Creating a shooting follower familiar

This code will automatically make the familiar a "follower" familiar, being inserted into the familiar line and following the player/the familiar in front of them in line.

```Lua
--MC_FAMILIAR_INIT passes the familiar its initializing.
function mod:FamiliarInit(familiar)
	--Register the follower as a familiar and intended to be part of the familiar line
	familiar:AddToFollowers()
end

--MC_FAMILIAR_INIT is the INIT callback for familiars. It accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_FAMILIAR_INIT, mod.FamiliarInit, FRIEND_FRANKIE_FAMILIAR)

--MC_FAMILIAR_UPDATE passes the familiar its updating.
function mod:FamiliarUpdate(familiar)
	--Familiar will follow the player/familiar in front of it
	familiar:FollowParent()
end

--MC_FAMILIAR_UPDATE is the UPDATE callback for familiars. It accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.FamiliarUpdate, FRIEND_FRANKIE_FAMILIAR)
```

:modding-repentogon: REPENTOGON also allows you to assign the priority of a familiar through [MC_GET_FOLLOWER_PRIORITY](https://repentogon.com/enums/ModCallbacks.html#mc_get_follower_priority). With [FollowerPriority.SHOOTER](https://repentogon.com/enums/FollowerPriority.html), the familiar is further back in the line, but is in front of any other familiars without an assigned priority. It also affects how the familiar is treated for Lilith and Tainted Lilith's Birthright.

```Lua
--Don't need to do anything other than return the new priority as it only runs for our familiar.
function mod:FamilarPriority()
	return FollowerPriority.SHOOTER
end

--Callback accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_GET_FOLLOWER_PRIORITY, mod.FamiliarPriority, FAMILIAR_VARIANT)
```

### Basic shooting familiar

The rest of this article will focus on creating a simple familiar like [Brother Bobby](https://bindingofisaacrebirth.wiki.gg/wiki/Brother_Bobby). There is a simple function that will handle the vast majority of work involved: [EntityFamiliar:Shoot()](https://wofsauge.github.io/IsaacDocs/rep/EntityFamiliar.html#shoot).

```Lua
function mod:FamiliarUpdate(familiar)
	familiar:Shoot()
	familiar:FollowParent()
end

mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.FamiliarUpdate, FRIEND_FRANKIE_FAMILIAR)
```

This function, when ran on `MC_FAMILIAR_UPDATE`, will make your familiar fire regular tears 1.36 times a second dealing 3.5 damage per shot. It will automatically synergize with familiar modifiers such as [Baby-Bender](https://bindingofisaacrebirth.wiki.gg/wiki/Baby-Bender), [BFFS!](https://bindingofisaacrebirth.wiki.gg/wiki/BFFS!), and [Forgotten Lullaby](https://bindingofisaacrebirth.wiki.gg/wiki/Forgotten_Lullaby), and firing direction overrides like [King Baby](https://bindingofisaacrebirth.wiki.gg/wiki/King_Baby) and [Marked](https://bindingofisaacrebirth.wiki.gg/wiki/Marked).

It will also automatically play certain animations in your familiar's anm2 file. You can view `003.001_brother bobby.anm2` inside the game's [extracted resources](creating_a_mod.md#extracting-the-games-resources) for reference.

It is possible to modify the firing speed and attributes of the tear. Without REPENTOGON, you will need to some manual checks inside `MC_FAMILIAR_UPDATE`. This code will alter the fire rate of the familiar so that it fires 1 tear per second, does double the normal damage, and slows enemies.

```Lua
--Define the desired FireCooldown for the familiar.
--30 is equivalent to 1 second in an update callback.
local NEW_COOLDOWN = 30
--By default, FireCooldown is set to 22 when firing a tear.
local DEFAULT_COOLDOWN = 22
--This is kept as a "percentage" of how much the cooldown is to be adjusted by.
--so that Forgotten Lullaby is accounted for with its adjustment to the default cooldown.
local COOLDOWN_MULT = NEW_COOLDOWN / DEFAULT_COOLDOWN

function mod:FamiliarUpdate(familiar)
	local player = familiar.Player
	local fireDir = player:GetFireDirection()
	--Check FireCooldown before :Shoot() adjusts it as familiars can only fire if its 0.
	--It's <= 1 specifically because :Shoot() will subtract FireCooldown on its own, check if its 0, then fire again if so.
	local canFire = familiar.FireCooldown <= 1

	familiar:Shoot()

	--Checking FireCooldown > 0, :Shoot() should have successfully fired a tear.
	if canFire and familiar.FireCooldown > 0 then
		--Set new cooldown. It's under math.floor to turn it into an integer, as FireCooldown doesn't accept floats.
		familiar.FireCooldown = math.floor(familiar.FireCooldown * COOLDOWN_MULT)

		--:Shoot() will already have spawned the tear. Search for the tear with Isaac.FindByType to make modifications to it.
		--MC_POST_TEAR_INIT isn't reliable as tear effects/damage are overridden afterwards.
		for _, ent in ipairs(Isaac.FindByType(EntityType.ENTITY_TEAR, TearVariant.BLUE, 0)) do
			--Check its only from our familiar by checking the Type and Variant of what spawned it.
			if ent.SpawnerType == EntityType.ENTITY_FAMILIAR
				and ent.SpawnerVariant == FRIEND_FRANKIE_FAMILIAR
				and tear.FrameCount == 0 --Check that it just spawned.
			then
				--Isaac.FindByType always passes an Entity object. Make it an EntityFamiliar to access :AddTearFlags().
				local tear = ent:ToTear()
				--Add tear modifiers.
				tear:AddTearFlags(TearFlags.TEAR_SLOW)
				tear.CollisionDamage = tear.CollisionDamage * 2
			end
		end
	end

	familiar:FollowParent()
end

mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.FamiliarUpdate, FRIEND_FRANKIE_FAMILIAR)
```

If using :modding-repentogon: REPENTOGON, you can instead use a callback named [MC_POST_FAMILIAR_FIRE_PROJECTILE](https://repentogon.com/enums/ModCallbacks.html#mc_post_familiar_fire_projectile) that triggers when `EntityFamiliar:Shoot` is called and a tear is fired. You can then adjust the properties of the tear. Note that modifications to FireCooldown will still need to be done within `MC_FAMILIAR_UPDATE`.

```Lua
--Callback only passes the fired tear
function mod:FamiliarShoot(tear)
	tear:AddTearFlags(TearFlags.TEAR_SLOW)
	tear.CollisionDamage = tear.CollisionDamage * 2
end

--Callback accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_POST_FAMILIAR_FIRE_PROJECTILE, mod.FamiliarShoot, FRIEND_FRANKIE_FAMILIAR)
```

As a final touch, :modding-repentogon: REPENTOGON also allows you to assign the priority of a familiar through [MC_GET_FOLLOWER_PRIORITY](https://repentogon.com/enums/ModCallbacks.html#mc_get_follower_priority). With [FollowerPriority.SHOOTER](https://repentogon.com/enums/FollowerPriority.html), the familiar is further back in the line, but is in front of any other familiars without an assigned priority. It also affects how the familiar is treated for Lilith and Tainted Lilith's birthright.

```Lua
--Don't need to do anything other than return the new priority as it only runs for our familiar.
function mod:FamilarPriority()
	return FollowerPriority.SHOOTER
end

--Callback accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_GET_FOLLOWER_PRIORITY, mod.FamiliarPriority, FRIEND_FRANKIE_FAMILIAR)
```

With that, your follower is complete!

<p align="center">
  <img src="../../assets/familiars/follower_final.gif" alt="Showcasing the shooter familiar" />
</p>

### Creating an orbital familiar

An orbital familiar will orbit around the player, circling around them at a set distance. For this orbital, we will create an orbital similarly to Cube of Meat, which will orbit around the player, block projectiles, and deal damage.

First, handling the basics of an orbital familiar. You will need to choose which layer to add your orbital to, which affects its distance, speed, and will change position based no how many familiars are on the same layer.

???- info "Orbital layer information"
	???+ note "Undefined layers"
		Familiars will have an orbit layer of `-1` by default. An orbital with a layer of `-1` or below will stick to the center of the player, while layers at or above `11` will have identical speed and distance as layer `10`.

	|Layer|Vanilla Familiars|Speed|Distance|
	|:--|:--|:--|:--|
	|0|[Cube of Meat](https://bindingofisaacrebirth.wiki.gg/wiki/Cube_of_Meat), [Ball of Bandages](https://bindingofisaacrebirth.wiki.gg/wiki/Ball_of_Bandages), [Guardian Angel](https://bindingofisaacrebirth.wiki.gg/wiki/Guardian_Angel), [Pretty Fly](https://bindingofisaacrebirth.wiki.gg/wiki/Familiar#Pretty_Fly), [Sacrificial Dagger](https://bindingofisaacrebirth.wiki.gg/wiki/Sacrificial_Dagger), [Slipped Rib](https://bindingofisaacrebirth.wiki.gg/wiki/Slipped_Rib), [Smart Fly](https://bindingofisaacrebirth.wiki.gg/wiki/Smart_Fly), [Sworn Protector](https://bindingofisaacrebirth.wiki.gg/wiki/Sworn_Protector), [Spin to Win](https://bindingofisaacrebirth.wiki.gg/wiki/Spin_to_Win)|0.045|X: 25, Y: 20|
	|1|[Blue Fly](https://bindingofisaacrebirth.wiki.gg/wiki/Familiar#Blue_Flies)|0.045|X: 35, Y: 30|
	|2|[Bot Fly](https://bindingofisaacrebirth.wiki.gg/wiki/Bot_Fly), [Distant Admiration](https://bindingofisaacrebirth.wiki.gg/wiki/Distant_Admiration)|-0.029|X: 60, Y: 45|
	|3|[Best Bud](https://bindingofisaacrebirth.wiki.gg/wiki/Best_Bud), [Forever Alone](https://bindingofisaacrebirth.wiki.gg/wiki/Forever_alone)|0.019|X: 110, Y: 90|
	|4|[Angelic Prism](https://bindingofisaacrebirth.wiki.gg/wiki/Angelic_Prism), [Friend Zone](https://bindingofisaacrebirth.wiki.gg/wiki/Friend_Zone)|0.019|X: 85, Y: 67.5|
	|5|[Big Fan](https://bindingofisaacrebirth.wiki.gg/wiki/Big_Fan), [Bloodshot Eye](https://bindingofisaacrebirth.wiki.gg/wiki/Bloodshot_Eye), [Guillotine](https://bindingofisaacrebirth.wiki.gg/wiki/Guillotine), [Leprosy](https://bindingofisaacrebirth.wiki.gg/wiki/Leprosy), [Mom's Razor](https://bindingofisaacrebirth.wiki.gg/wiki/Mom%27s_Razor), [Book of the Dead's bone orbitals](https://bindingofisaacrebirth.wiki.gg/wiki/Book_of_the_Dead)|0.045|X: 35, Y: 30|
	|6|[Psy Fly](https://bindingofisaacrebirth.wiki.gg/wiki/Psy_Fly), [Tinytoma](https://bindingofisaacrebirth.wiki.gg/wiki/Tinytoma)|0.029|X: 50, Y: 42|
	|7|[Lemegeton Wisps](https://bindingofisaacrebirth.wiki.gg/wiki/Lemegeton) (Layer 1)|0.039|X: 28, Y: 21|
	|8|[Lemegeton Wisps](https://bindingofisaacrebirth.wiki.gg/wiki/Lemegeton) (Layer 2)|-0.029|c|
	|9|[Lemegeton Wisps](https://bindingofisaacrebirth.wiki.gg/wiki/Lemegeton) (Layer 3)|0.019|X: 64, Y: 50|
	|10|[Swarm Fly](https://bindingofisaacrebirth.wiki.gg/wiki/The_Swarm)|0.019|X: 40, Y: 36|

Speed determines how fast the orbital travels in its orbit each game tick, as well as the direction. A positive speed indicates counter-clockwise, while a negative speed makes the orbital travel clockwise.

Distance is determined by a Vector, where X is the farthest left/right position and Y is the highest/lowest point of the orbit. Below is a visualization using Angelic Prism:

<p align="center">
  <img src="../../assets/familiars/orbital_distance.png" alt="Visual representation of orbit distance" />
</p>

These attributes will be automatically applied to the orbital when using [EntityFamiliar:AddToLayer](https://wofsauge.github.io/IsaacDocs/rep/EntityFamiliar.html#addtolayer), so there is no need to set those yourself.

```Lua
--MC_FAMILIAR_INIT passes the familiar its initializing.
function mod:FamiliarInit(familiar)
	--Register the follower as an orbital to be part of the assigned orbit layer
	familiar:AddToOrbit(0)
end

--MC_FAMILIAR_INIT is the INIT callback for familiars. It accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_FAMILIAR_INIT, mod.FamiliarInit, FRIEND_FRANKIE_FAMILIAR)
```

Now that the familiar is registered as an orbital, it needs to follow its expected orbital position. [EntityFamiliar:GetOrbitPosition](https://wofsauge.github.io/IsaacDocs/rep/EntityFamiliar.html#getorbitposition) will return the expected orbit position of the familiar, but the position should be translated into velocity as to not have snappy movement.

```Lua
--MC_FAMILIAR_UPDATE passes the familiar its updating.
function mod:FamiliarUpdate(familiar)
	--The position passed into GetOrbitPosition will act as the center position the familiar orbits around.
	local orbitPos = familiar:GetOrbitPosition(familiar.Player.Position)
	--Set familar velocity
	familiar.Velocity = orbitPos - familiar.Position
end

--MC_FAMILIAR_UPDATE is the UPDATE callback for familiars. It accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.FamiliarUpdate, FRIEND_FRANKIE_FAMILIAR)
```

Next, contact damage. This is trivial to create, as you just need your [entities2.xml](entity_basics.md#entity-tag) file. In your familiar entry, you can update `collisionDamage` and `collisionInterval` for how much damage the familiar deals and how often it deals it. `collisionDamage="7"` and `collisionInterval="4"` are the exact values that Cube of Meat uses.

```XML
<entities anm2root="gfx/" version="5">
    <entity name="Friend Frankie" id="3" variant="508" anm2path="familiar_friend_frankie.anm2"
    tags="cansacrifice" shadowSize="11" friction="1" collisionDamage="7" collisionInterval="4" />
</entities>
```

For blocking projectiles, there are different methods depending on usage of REPENTOGON.

With :modding-repentogon: REPENTOGON, create a [customtags](https://repentogon.com/xml/entities.html#customtags) variable and add `familiarblockprojectiles`, like so:

```XML
<entities anm2root="gfx/" version="5">
    <entity name="Friend Frankie" id="3" variant="508" anm2path="familiar_friend_frankie.anm2"
    tags="cansacrifice" shadowSize="11" friction="1" collisionDamage="7" collisionInterval="4" customtags="familiarblockprojectiles" />
</entities>
```

Without REPENTOGON, you will need to detect projectiles on collision and destroy them there. Use [MC_PRE_FAMILIAR_COLLISION](https://wofsauge.github.io/IsaacDocs/rep/enums/ModCallbacks.html#mc_pre_familiar_collision) to accomplish this, and [Entity:Die](https://wofsauge.github.io/IsaacDocs/rep/Entity.html#die) to kill the projectile.

```Lua
function mod:FamiliarCollision(familiar, collider)
	if collider.Type == EntityType.ENTITY_PROJECTILE then
		collider:Die()
	end
end

--Adds a priority callback on CallbackPriority.LATE in case other mods may intentionally cancel collision earlier in the callback.
--Due to the lack of a MC_POST_FAMILIAR_COLLISION, this is the best that can be done to estimate if the projectile is expected to collide with the familiar.
mod:AddPriorityCallback(ModCallbacks.MC_PRE_FAMILIAR_COLLISION, CallbackPriority.LATE, mod.FamiliarCollision, FRIEND_FRANKIE_FAMILIAR)
```

With that, your orbital is finished!

<p align="center">
  <img src="../../assets/familiars/orbital_final.gif" alt="Showcasing the orbital familiar" />
</p>

### Creating a delayed familiar

Delayed familiars are familiars that follow the player's exact movements on a specific delay. Adding one is fairly simple: Call [EntityFamiliar:AddToDelayed](https://wofsauge.github.io/IsaacDocs/rep/EntityFamiliar.html#addtodelayed) on `MC_FAMILIAR_INIT` and [EntityFamiliar:MoveDelayed](https://wofsauge.github.io/IsaacDocs/rep/EntityFamiliar.html#movedelayed) on `MC_FAMILIAR_UPDATE` with the number of frames the movement should be delayed relative to its parent. The first familiar will delay its movements relative to the player's movements, while extra familiars will delay it relative to the last familiar's movements.

```Lua
local NUM_DELAY_FRAMES = 30

--MC_FAMILIAR_INIT passes the familiar its initializing.
function mod:FamiliarInit(familiar)
	--Register the delayed familiar.
	familiar:AddToDelayed()
end

--MC_FAMILIAR_INIT is the INIT callback for familiars. It accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_FAMILIAR_INIT, mod.FamiliarInit, FRIEND_FRANKIE_FAMILIAR)

--MC_FAMILIAR_UPDATE passes the familiar its updating.
function mod:FamiliarUpdate(familiar)
	familiar:MoveDelayed(NUM_DELAY_FRAMES)
end

--MC_FAMILIAR_UPDATE is the UPDATE callback for familiars. It accepts an optional argument to only run for your familiar variant.
mod:AddCallback(ModCallbacks.MC_FAMILIAR_UPDATE, mod.FamiliarUpdate, FRIEND_FRANKIE_FAMILIAR)
```

<p align="center">
  <img src="../../assets/familiars/delayed_final.gif" alt="Showcasing the delayed familiar" />
</p>
